# JavaScript & DOM — Deep Reference

This is not an introduction. It assumes you are writing handwritten JS for a static site and need to understand not just what the browser does but why, and what the cost is.

---

## 1. The Browser Rendering Pipeline — Real Depth

### The Full Pipeline

```
JavaScript → Style → Layout → Paint → Composite
```

Each stage has a cost. Skipping stages is always better than running them. Understanding which operations touch which stages is the difference between smooth and janky.

**Style** — The browser recalculates which CSS rules apply to which elements. Triggered by class changes, attribute changes, style mutations. Cost scales with selector complexity and number of affected elements.

**Layout (Reflow)** — The browser computes geometry: where every element is, how large it is, how it affects its neighbours. This is expensive. It is the stage you most want to avoid triggering unnecessarily. Layout is always at least partially global — changing one element's geometry can force recalculation of its ancestors, descendants, and siblings.

**Paint** — The browser fills in pixels for each layer. Text rendering, borders, shadows, backgrounds. Not all elements repaint when the layout changes — only those in affected layers.

**Composite** — The GPU assembles layers into the final frame. This is the cheapest stage. Operations that only affect compositing — `transform`, `opacity` — skip Layout and Paint entirely. This is why they are the correct properties for animation.

### Compositor Layers — What They Are and When They Exist

A compositor layer is a texture uploaded to the GPU. The browser creates them automatically in some cases and you can hint for them explicitly.

**Conditions that create a new stacking context** (and often a compositor layer):
- `position: fixed` or `position: sticky`
- `opacity` less than 1
- `transform` other than `none`
- `will-change` set to a compositable property
- `isolation: isolate`
- `filter` or `backdrop-filter`
- `mix-blend-mode` other than `normal`

**`will-change`** tells the browser to promote an element to its own layer before the animation starts, avoiding a promotion cost mid-animation. Use it deliberately:
```css
.panel {
  will-change: transform; /* promote before needed */
}
```
Remove it after the animation is done — each layer consumes GPU memory. Layer explosion (dozens of promoted elements) is a real problem. Check the Layers panel in DevTools to see what's been promoted and how much memory each costs.

### Layout Thrash — The Most Common Expensive Mistake

Reading a geometric property after a DOM write forces the browser to flush its pending layout queue synchronously. This is layout thrash.

**Properties that force synchronous layout when read:**
`offsetWidth`, `offsetHeight`, `offsetTop`, `offsetLeft`, `offsetParent`, `scrollTop`, `scrollLeft`, `scrollWidth`, `scrollHeight`, `clientWidth`, `clientHeight`, `clientTop`, `clientLeft`, `getBoundingClientRect()`, `getComputedStyle()`, `innerText` (triggers layout to compute line breaks).

**The pattern:**
```js
// BAD — interleaved reads and writes
elements.forEach(el => {
  const h = el.offsetHeight;       // read → forced reflow
  el.style.height = h + 10 + 'px'; // write → invalidates layout
});
// Each iteration forces a full reflow. N elements = N reflows.

// GOOD — batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // all reads → one reflow
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px';           // all writes
});
// One reflow total.
```

Use `DocumentFragment` for bulk DOM insertion — build the entire subtree off-document, then insert once.

### Animating Correctly — What the Compositor Can Handle

Only two CSS properties are fully composited — they skip Layout and Paint:
- `transform` (translate, scale, rotate, skew, matrix)
- `opacity`

Everything else — `width`, `height`, `top`, `left`, `margin`, `padding`, `background-color` — triggers Layout or Paint when animated. Express motion with `transform`. Express fade with `opacity`. That is the rule.

```js
// BAD — triggers layout on every frame
el.style.left = x + 'px';

// GOOD — compositor only
el.style.transform = `translateX(${x}px)`;
```

---

## 2. The JavaScript Engine — V8 and JavaScriptCore

### How Engines Optimise Code

Modern JS engines (V8 in Chrome/Node, JavaScriptCore in Safari/WebKit) use a JIT (Just-In-Time) compiler. Code starts interpreted, then hot paths get compiled to optimised machine code.

**Hidden classes** — V8 tracks the "shape" of objects (which properties they have, in what order). Objects with the same shape share a hidden class and get optimised together.

```js
// GOOD — consistent shape, same hidden class
function Point(x, y) {
  this.x = x;
  this.y = y;
}

// BAD — different shapes, different hidden classes, no optimisation sharing
const a = { x: 1, y: 2 };
const b = { x: 1, y: 2, z: 3 }; // different shape
```

Always initialise all properties in the constructor, in the same order. Adding properties dynamically after construction creates new hidden classes and deoptimises.

**Inline caches** — the engine caches the type of a property lookup. If a function always receives the same shaped object, the lookup is cached and fast. If it receives differently shaped objects, the cache becomes polymorphic (slower) or megamorphic (much slower, no cache).

**Deoptimisation** — the engine can bail out of optimised code back to interpreted mode if its assumptions are violated. Common causes: changing an object's shape after compilation, changing a variable's type (number → string), using `arguments` object, `try/catch` around hot code.

### Garbage Collection — Not Triggering It

GC pauses cause jank. In animation loops and scroll handlers, avoid allocation:

```js
// BAD — allocates a new object every frame
function update() {
  const pos = { x: mouse.x, y: mouse.y }; // allocation
  draw(pos);
  requestAnimationFrame(update);
}

// GOOD — reuse the same object
const pos = { x: 0, y: 0 };
function update() {
  pos.x = mouse.x;
  pos.y = mouse.y;
  draw(pos);
  requestAnimationFrame(update);
}
```

Avoid creating closures, arrays, or objects inside hot loops. Pre-allocate where possible. Use typed arrays (`Float32Array`, `Int32Array`) for numerical data in animation — they have fixed memory layout and no GC pressure.

---

## 3. The Event Loop — Precise Understanding

### The Full Model

The JavaScript runtime has:
- **Call stack** — currently executing code
- **Microtask queue** — Promise callbacks, `queueMicrotask()`, `MutationObserver` callbacks
- **Task queue (macrotask queue)** — `setTimeout`, `setInterval`, I/O, UI events
- **Rendering** — happens between tasks, not between microtasks

**The order:**
1. Execute the current task (call stack empties)
2. Drain the entire microtask queue (all microtasks, including any queued by microtasks)
3. Render if needed (style, layout, paint, composite)
4. Pick the next task from the task queue
5. Repeat

This is why Promise callbacks always fire before `setTimeout(fn, 0)`:

```js
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));

// Output:
// promise   ← microtask, runs before next task
// timeout   ← macrotask, runs after
```

### Microtasks vs Macrotasks — When It Matters

Microtasks run to completion before rendering. If you queue microtasks in a loop, rendering is blocked until all of them are done. This can cause visual freezes even though no single microtask is slow.

```js
// This blocks rendering — all microtasks run before the browser can paint
function infiniteMicrotask() {
  Promise.resolve().then(infiniteMicrotask);
}
```

Use `setTimeout(fn, 0)` or `requestAnimationFrame` to yield to the renderer. Use `requestIdleCallback` for non-urgent work.

### `queueMicrotask`
Explicit microtask scheduling without a Promise:
```js
queueMicrotask(() => {
  // runs after current script, before next task, before rendering
});
```

Useful for deferring work to the end of the current synchronous block without the overhead of a Promise.

### `requestAnimationFrame` — Not Just "Before Draw"

`rAF` fires after the browser has processed input events but before paint. It is the correct place for any visual update. It is synchronised to the display refresh rate (typically 60Hz, 120Hz on ProMotion displays).

```js
let frameId;
let lastTimestamp;

function loop(timestamp) {
  // timestamp is DOMHighResTimeStamp — milliseconds with sub-millisecond precision
  const delta = lastTimestamp ? timestamp - lastTimestamp : 0;
  lastTimestamp = timestamp;

  // use delta for frame-rate-independent animation
  position += velocity * delta;

  draw();
  frameId = requestAnimationFrame(loop);
}

frameId = requestAnimationFrame(loop);

// Always cancel when done — orphaned rAF loops run forever
cancelAnimationFrame(frameId);
```

**Frame-rate independence is not optional.** A user on a 120Hz display gets twice as many frames. Velocity and physics must be multiplied by delta, not assumed to run at 60fps.

### `requestIdleCallback`
Schedules work during browser idle time — after rendering, while waiting for the next input or frame.

```js
requestIdleCallback((deadline) => {
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    tasks.shift()(); // do a unit of work
  }
  if (tasks.length > 0) {
    requestIdleCallback(processWork); // schedule next chunk
  }
});
```

Use for: preloading, analytics, indexing, anything that doesn't need to happen immediately. Never use for rendering or interaction response.

---

## 4. DOM — Surgical Knowledge

### Selection — Live vs Static

```js
querySelectorAll('.item')        // static NodeList — snapshot at call time
getElementsByClassName('item')  // live HTMLCollection — reflects DOM changes
getElementsByTagName('div')      // live HTMLCollection
```

Live collections can cause infinite loops if you add matching elements while iterating. Static NodeLists are safer for most use cases. Convert to Array when you need array methods: `Array.from(nodeList)` or `[...nodeList]`.

### Traversal
```js
el.closest('.ancestor')         // walks up; returns null if not found
el.matches('.selector')         // tests this element; useful in delegation
el.contains(otherEl)            // true if otherEl is a descendant
el.children                     // live HTMLCollection of element children only
el.childNodes                   // live NodeList of all nodes (incl. text, comments)
el.parentElement                // nearest element ancestor
el.nextElementSibling           // next sibling that is an element
el.previousElementSibling
```

### Insertion — Know the Difference
```js
parent.appendChild(el)           // appends at end; returns el
parent.prepend(el)               // inserts at start; accepts multiple nodes/strings
parent.append(el, 'text', el2)  // appends; accepts multiple nodes/strings
parent.insertBefore(el, ref)    // inserts before ref node
ref.insertAdjacentElement('beforebegin', el) // sibling before ref
ref.insertAdjacentElement('afterbegin', el)  // first child of ref
ref.insertAdjacentElement('beforeend', el)   // last child of ref
ref.insertAdjacentElement('afterend', el)    // sibling after ref
ref.insertAdjacentHTML('beforeend', '<p>text</p>') // parses and inserts HTML
```

`insertAdjacentHTML` does not destroy existing siblings or their event listeners. Prefer over `innerHTML` for partial updates.

### DocumentFragment — Batch Insertion
```js
const fragment = document.createDocumentFragment();
items.forEach(item => {
  const el = document.createElement('div');
  el.textContent = item.name;
  fragment.appendChild(el); // off-document, no reflow
});
container.appendChild(fragment); // one reflow
```

### dataset — HTML as State Carrier
```js
el.dataset.state = 'open';     // sets data-state="open"
el.dataset.projectId = '42';   // sets data-project-id="42"
delete el.dataset.state;       // removes data-state

// Target in CSS
[data-state="open"] { display: block; }
```

State stored in `dataset` is visible in DevTools, inspectable, and directly targetable from CSS without JS involvement. For static sites this is often the right state management pattern.

### Geometry — What Each Property Actually Returns

```js
el.getBoundingClientRect()
// DOMRect: { top, right, bottom, left, width, height, x, y }
// All relative to the viewport. Includes transforms.
// Triggers layout. Cache if reading in a loop.

el.offsetWidth / el.offsetHeight
// Integer pixel dimensions, excluding margins, including border and padding.
// Does NOT account for CSS transforms.
// Triggers layout.

el.clientWidth / el.clientHeight
// Dimensions excluding border and scrollbar, including padding.
// Triggers layout.

el.scrollWidth / el.scrollHeight
// Total scrollable dimensions including content outside the visible area.

window.scrollY / window.scrollX
// Current document scroll position. No layout trigger on read alone.

el.scrollTop / el.scrollLeft
// Scroll offset within a scrollable element.
```

**`getComputedStyle(el)`** — returns the resolved, used values of CSS properties after cascade and inheritance. This is a "resolved value" — different from what you set. `width: auto` computes to a pixel value. Triggers layout.

```js
const style = getComputedStyle(el);
const fontSize = parseFloat(style.fontSize); // in px regardless of how it was set
```

**`getBoundingClientRect` vs `offsetTop`** — `getBoundingClientRect` is relative to the viewport and accounts for transforms. `offsetTop` is relative to `offsetParent` and ignores transforms. For anything involving scroll position or transforms, use `getBoundingClientRect`.

---

## 5. Events — Full Depth

### Propagation Model
```
Capture phase: window → document → <html> → <body> → ... → target
Target phase:  target
Bubble phase:  target → ... → <body> → <html> → document → window
```

Almost all events bubble. Notable exceptions: `focus`, `blur`, `load`, `unload`, `scroll` (on elements). Use `focusin`/`focusout` if you need bubbling focus events.

```js
// Capture phase listener — fires before bubble phase
el.addEventListener('click', handler, { capture: true });
// or
el.addEventListener('click', handler, true);
```

### addEventListener Options
```js
el.addEventListener('click', handler, {
  capture: false,  // bubble phase (default)
  once: true,      // automatically removes after first call
  passive: true,   // promises not to call preventDefault; allows browser scroll optimisation
  signal: controller.signal // tied to AbortController
});
```

**`passive: true`** is critical for scroll and touch listeners. Without it, the browser waits to see if you'll call `preventDefault` before scrolling — introducing input latency. If you don't need to cancel the event, always pass `passive: true`.

```js
window.addEventListener('scroll', handler, { passive: true });
window.addEventListener('touchstart', handler, { passive: true });
```

### Event Delegation — Correct Pattern
```js
document.querySelector('.list').addEventListener('click', (e) => {
  // e.target is the actual clicked element
  // e.currentTarget is .list — where the listener lives
  const item = e.target.closest('[data-item-id]');
  if (!item) return; // click was on the container, not an item
  handleItem(item.dataset.itemId);
});
```

`closest` is the correct tool here — it handles clicks on child elements of the item gracefully.

### Custom Events — Module Communication
```js
// Dispatch — bubbles up the tree
el.dispatchEvent(new CustomEvent('panel:open', {
  bubbles: true,
  composed: true, // crosses shadow DOM boundaries if needed
  detail: { id: 'project-1', source: 'keyboard' }
}));

// Listen at document level — catches all dispatched events
document.addEventListener('panel:open', ({ detail }) => {
  openPanel(detail.id);
});
```

Custom events with `bubbles: true` are a clean inter-module communication system. No shared state required. Use namespaced names (`module:action`) to avoid collisions.

### AbortController — Listener Lifecycle
```js
class PanelManager {
  #controller = null;

  mount() {
    this.#controller = new AbortController();
    const { signal } = this.#controller;

    document.addEventListener('keydown', this.#handleKey, { signal });
    window.addEventListener('resize', this.#handleResize, { signal, passive: true });
    this.#el.addEventListener('click', this.#handleClick, { signal });
  }

  unmount() {
    this.#controller?.abort(); // removes all listeners at once
    this.#controller = null;
  }
}
```

One `abort()` call removes every listener that shares the signal. No manual `removeEventListener` bookkeeping.

### Pointer Events — Unified Input
```js
// Replaces mousedown + touchstart + pointerdown as appropriate
el.addEventListener('pointerdown', handler);
el.addEventListener('pointermove', handler);
el.addEventListener('pointerup', handler);
el.addEventListener('pointercancel', handler);

// Capture pointer on drag — events continue even if cursor leaves element
el.addEventListener('pointerdown', (e) => {
  el.setPointerCapture(e.pointerId);
});
```

`pointerType` tells you whether the event came from `'mouse'`, `'touch'`, or `'pen'`.

---

## 6. Observer APIs

### IntersectionObserver — Precise Usage

```js
const io = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // entry.isIntersecting — boolean
    // entry.intersectionRatio — 0 to 1, how much is visible
    // entry.boundingClientRect — element's rect
    // entry.rootBounds — root's rect (viewport if no root set)
    // entry.time — timestamp of the intersection change
    entry.target.classList.toggle('visible', entry.isIntersecting);
  });
}, {
  root: null,                      // null = viewport
  rootMargin: '0px 0px -10% 0px', // shrinks effective viewport from bottom
  threshold: [0, 0.25, 0.5, 1]    // fire at each of these ratios
});

// Observe multiple elements
document.querySelectorAll('.reveal').forEach(el => io.observe(el));

// Unobserve a single element
io.unobserve(el);

// Disconnect entirely
io.disconnect();
```

**`rootMargin`** accepts negative values to shrink the intersection root — useful for triggering animations before the element fully enters, or only after it's well inside the viewport. Works like CSS margin: top right bottom left.

**Multiple thresholds** let you respond to scroll progress, not just in/out state. At `threshold: [0, 0.25, 0.5, 0.75, 1]` you get five callbacks as an element scrolls into view.

### ResizeObserver — Box Model Detail

```js
const ro = new ResizeObserver((entries) => {
  for (const entry of entries) {
    // entry.contentRect — content area (excludes padding, border)
    // entry.borderBoxSize — array of { blockSize, inlineSize } (includes border+padding)
    // entry.contentBoxSize — array of { blockSize, inlineSize } (content only)
    // entry.devicePixelContentBoxSize — physical pixels; critical for canvas

    const { inlineSize: width, blockSize: height } = entry.contentBoxSize[0];
    // inlineSize = width in horizontal writing modes
    // blockSize = height in horizontal writing modes
  }
});

ro.observe(el);
ro.observe(el, { box: 'border-box' }); // report border-box dimensions
```

**`devicePixelContentBoxSize`** gives you the exact physical pixel dimensions of the content box. Use this for canvas sizing to avoid blurriness on high-DPI displays:

```js
const ro = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const [{ inlineSize: w, blockSize: h }] = entry.devicePixelContentBoxSize;
    canvas.width = w;
    canvas.height = h;
    // don't touch canvas.style.width/height — CSS controls display size
    draw();
  }
});
ro.observe(canvas, { box: 'device-pixel-content-box' });
```

### MutationObserver

```js
const mo = new MutationObserver((mutations) => {
  for (const mutation of mutations) {
    if (mutation.type === 'childList') {
      mutation.addedNodes.forEach(node => { /* node was inserted */ });
      mutation.removedNodes.forEach(node => { /* node was removed */ });
    }
    if (mutation.type === 'attributes') {
      // mutation.attributeName — which attribute changed
      // mutation.oldValue — previous value (if attributeOldValue: true)
      const newValue = mutation.target.getAttribute(mutation.attributeName);
    }
    if (mutation.type === 'characterData') {
      // text content changed
    }
  }
});

mo.observe(el, {
  childList: true,           // watch for child additions/removals
  subtree: true,             // watch entire subtree, not just direct children
  attributes: true,          // watch attribute changes
  attributeFilter: ['data-state', 'aria-expanded'], // only these attributes
  attributeOldValue: true,   // include previous value in mutation record
  characterData: true,       // watch text node changes
  characterDataOldValue: true
});

mo.disconnect(); // stop observing
```

---

## 7. Animation — Complete

### requestAnimationFrame — Correct Patterns

```js
// Frame-rate independent animation
const state = { x: 0, velocity: 200 }; // 200px per second
let lastTime = null;
let frameId = null;

function tick(now) {
  if (lastTime !== null) {
    const delta = (now - lastTime) / 1000; // convert ms to seconds
    state.x += state.velocity * delta;
  }
  lastTime = now;

  el.style.transform = `translateX(${state.x}px)`;
  frameId = requestAnimationFrame(tick);
}

function start() { frameId = requestAnimationFrame(tick); }
function stop() { cancelAnimationFrame(frameId); lastTime = null; }
```

### Web Animations API — Full Control

```js
const anim = el.animate([
  { transform: 'translateY(24px)', opacity: 0 },
  { transform: 'translateY(0)', opacity: 1 }
], {
  duration: 400,
  delay: 0,
  endDelay: 0,
  easing: 'cubic-bezier(0.25, 0.46, 0.45, 0.94)',
  fill: 'forwards',     // retain end state after finish
  iterations: 1,        // Infinity for looping
  direction: 'normal',  // 'reverse', 'alternate', 'alternate-reverse'
  composite: 'replace'  // 'add', 'accumulate'
});

// Control
anim.pause();
anim.play();
anim.reverse();
anim.finish();    // jumps to end
anim.cancel();    // removes effect and resets

// Timing
anim.currentTime = 200; // seek to 200ms
anim.playbackRate = 2;  // double speed

// Events
anim.finished.then(() => { /* done */ });
anim.addEventListener('finish', handler);
anim.addEventListener('cancel', handler);
```

**`getAnimations()`** — returns all active animations on an element. Useful for pausing all animations during reduced-motion scenarios.

### Respecting User Preferences

```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');

function animate(el) {
  if (prefersReducedMotion.matches) {
    el.classList.add('visible'); // instant state change, no animation
    return;
  }
  el.animate([...], { duration: 400 });
}

// React to preference changes dynamically
prefersReducedMotion.addEventListener('change', (e) => {
  if (e.matches) cancelAllAnimations();
});
```

### Scroll-Linked Animation — Two Approaches

**Throttled scroll listener** (maximum control):
```js
let ticking = false;
window.addEventListener('scroll', () => {
  if (ticking) return;
  requestAnimationFrame(() => {
    const progress = window.scrollY / (document.body.scrollHeight - window.innerHeight);
    el.style.setProperty('--scroll-progress', progress);
    ticking = false;
  });
  ticking = true;
}, { passive: true });
```

**CSS `animation-timeline: scroll()`** (zero JS, growing support):
```css
@keyframes reveal {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.section {
  animation: reveal linear;
  animation-timeline: scroll();
  animation-range: entry 0% entry 30%;
}
```
Check Can I Use before using. As of 2025, well-supported in Chrome/Edge, good in Firefox, Safari improving.

---

## 8. Asynchronous Patterns for Static Sites

Static sites primarily fetch two things: content data (`content.json`) and assets (images loaded lazily). Keep async simple.

### Fetch with AbortController
```js
async function loadContent(path) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 5000);

  try {
    const res = await fetch(path, { signal: controller.signal });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    if (err.name === 'AbortError') console.warn('Request timed out');
    else throw err;
  } finally {
    clearTimeout(timeout);
  }
}
```

### Dynamic Import — Code Splitting Without a Bundler
```js
// Load heavy module only when needed
async function openMap() {
  const { initMap } = await import('./map.js');
  initMap(container);
}
```

Works natively in modern browsers. With esbuild, produces a separate chunk automatically.

---

## 9. Module Architecture for Static Sites

### ES Module Patterns

```js
// state.js — single source of truth
const _state = {
  activeProject: null,
  menuOpen: false,
};

export const state = new Proxy(_state, {
  set(target, key, value) {
    target[key] = value;
    document.dispatchEvent(new CustomEvent(`state:${key}`, {
      bubbles: false,
      detail: { value, previous: target[key] }
    }));
    return true;
  }
});
```

```js
// main.js — wiring
import { state } from './state.js';
import { initNav } from './nav.js';
import { initProjects } from './projects.js';

document.addEventListener('DOMContentLoaded', () => {
  initNav();
  initProjects();
});
```

### Private Class Fields — Encapsulation Without Closures

```js
class Carousel {
  #el;
  #items;
  #currentIndex = 0;
  #controller = null;

  constructor(el) {
    this.#el = el;
    this.#items = [...el.querySelectorAll('.carousel-item')];
    this.#mount();
  }

  #mount() {
    this.#controller = new AbortController();
    const { signal } = this.#controller;
    this.#el.addEventListener('click', this.#handleClick, { signal });
  }

  #handleClick = (e) => { /* arrow fn preserves this */ };

  destroy() {
    this.#controller?.abort();
  }
}
```

Private fields (`#field`) are truly private — not accessible outside the class, not on the prototype. Not a convention (`_field`), an enforcement.

---

## 10. Web APIs — Edges and Details

### History API
```js
// Push a new URL without reload
history.pushState({ projectId: 'fond-bank' }, '', '/work/fond-bank');

// Replace current entry
history.replaceState({ projectId: 'fond-bank' }, '', '/work/fond-bank');

// Listen for back/forward
window.addEventListener('popstate', (e) => {
  const { projectId } = e.state ?? {};
  if (projectId) loadProject(projectId);
});
```

`pushState` does not fire `popstate`. Only browser navigation (back/forward) fires it.

### matchMedia — System Preferences in JS
```js
const dark = window.matchMedia('(prefers-color-scheme: dark)');
const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
const coarsePointer = window.matchMedia('(pointer: coarse)'); // touch device

// Query current state
if (dark.matches) applyDarkTheme();

// React to changes
dark.addEventListener('change', (e) => {
  e.matches ? applyDarkTheme() : applyLightTheme();
});
```

### Canvas — High-DPI Correct Setup
```js
function setupCanvas(canvas) {
  const dpr = window.devicePixelRatio ?? 1;
  const rect = canvas.getBoundingClientRect();

  // Physical pixel dimensions
  canvas.width = Math.round(rect.width * dpr);
  canvas.height = Math.round(rect.height * dpr);

  const ctx = canvas.getContext('2d');
  ctx.scale(dpr, dpr); // scale all drawing operations

  return ctx;
}
// After this, draw in CSS pixels — the scale handles physical pixels
```

Use `ResizeObserver` with `devicePixelContentBoxSize` to keep this correct when the element resizes or the user moves the window to a display with a different DPR.

### Intersection Observer — rootMargin Precision
`rootMargin` modifies the bounding box of the root for intersection purposes. It does not affect the observed element. Negative values shrink the effective viewport; positive values expand it.

```js
// Fire when element is at least 100px into the viewport from the bottom
{ rootMargin: '0px 0px -100px 0px' }

// Fire when element is within 200px of entering the viewport (preload use case)
{ rootMargin: '200px 0px 0px 0px' }
```

### CSS Custom Properties from JS
```js
// Write
el.style.setProperty('--offset', `${value}px`);
document.documentElement.style.setProperty('--theme-hue', '220');

// Read
const value = getComputedStyle(el).getPropertyValue('--offset').trim();

// Remove
el.style.removeProperty('--offset');
```

Custom properties set on `:root` (via `document.documentElement`) are available everywhere. Properties set on a specific element are scoped to that element and its descendants. This is the correct bridge between JS state and CSS visual response.

---

## 11. Performance Discipline

### DevTools — What to Actually Look At

**Performance tab:** Record a 3-second interaction. Look at:
- Long tasks (red bar at top) — anything over 50ms on the main thread
- Layout events (purple) — how often and how expensive
- Paint events (green) — what's repainting
- Composite layers (the Layers panel) — what's promoted and why

**Layers panel** (More Tools → Layers): see every compositor layer, its memory cost, and why it was created ("Has a will-change hint", "Is the root layer", etc.).

**Rendering tab** (More Tools → Rendering):
- Paint flashing — highlights areas repainting in green. If the whole page flashes green on scroll, something is wrong.
- Layout shift regions — highlights CLS-causing elements.
- Frame rendering stats — current fps overlay.

### Memory — Not Causing Leaks

Common leak patterns in vanilla JS:
- Event listeners on elements that are removed from the DOM but still referenced by a closure
- Timers (`setInterval`) that are never cleared
- `rAF` loops that are never cancelled
- Global variables accumulating data
- Event listeners on `window` or `document` from components that are "unmounted"

Use `AbortController` for listener cleanup. Cancel `rAF` on stop. Clear intervals. Avoid storing large data in module-level variables.

### Script Loading Strategy
```html
<!-- Render-blocking — avoid unless absolutely critical for initial render -->
<script src="critical.js"></script>

<!-- Deferred — executes after HTML is parsed, in order, before DOMContentLoaded -->
<script defer src="main.js"></script>

<!-- Async — executes as soon as downloaded, out of order, don't use for modules that depend on each other -->
<script async src="analytics.js"></script>

<!-- Module — deferred by default, strict mode, own scope -->
<script type="module" src="app.js"></script>
```

For a static site: `type="module"` for your own code (deferred automatically), `async` only for truly independent third-party scripts.

---

## 12. Security

**XSS — never trust content in innerHTML:**
```js
// UNSAFE — if name comes from any external source
el.innerHTML = `<h1>${name}</h1>`;

// SAFE
const h1 = document.createElement('h1');
h1.textContent = name; // textContent always treats value as text, never HTML
el.appendChild(h1);
```

**Sanitise HTML you must render:**
```js
// Native Sanitizer API (modern browsers)
const sanitizer = new Sanitizer();
el.setHTML(untrustedHTML, { sanitizer });

// Or DOMPurify (library, wide support)
el.innerHTML = DOMPurify.sanitize(untrustedHTML);
```

Never use inline handlers (`onclick="..."`). Never put untrusted content in `href`, `src`, or `style` attributes without validation. Never use `eval()`.
