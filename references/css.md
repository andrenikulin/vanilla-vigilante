# CSS — Deep Reference

CSS is not a collection of tricks. It is a system with precise rules about how values are resolved, how layout is computed, and how the cascade determines what applies. Working with it well means understanding those rules, not memorising workarounds for when they're misunderstood.

---

## 1. How CSS Actually Works — The Cascade and Value Resolution

### The Cascade — Real Order of Priority

When multiple declarations target the same property on the same element, the cascade resolves which one wins. Priority order, highest to lowest:

1. **Transition declarations** — actively transitioning values
2. **Important declarations** — `!important`, in reverse origin order
3. **Animation declarations** — actively animating values
4. **Normal declarations** — in origin order: author → user → user-agent

Within normal author declarations:
1. **Cascade layers** — later layer wins; unlayered CSS beats all layers
2. **Specificity** — higher specificity wins
3. **Order of appearance** — later declaration wins

### Specificity — Calculated, Not Guessed

Specificity is a three-number tuple: `(ID, CLASS, TYPE)`

| Selector | Specificity |
|---|---|
| `*` | 0-0-0 |
| `div` | 0-0-1 |
| `.item` | 0-1-0 |
| `div.item` | 0-1-1 |
| `#hero` | 1-0-0 |
| `[type="text"]` | 0-1-0 |
| `:hover` | 0-1-0 |
| `::before` | 0-0-1 |
| `:is(h1, h2)` | takes specificity of most specific argument |
| `:where(h1, h2)` | always 0-0-0 |

Numbers don't carry over — `0-1-0` beats `0-0-100`. IDs dominate. Avoid IDs in stylesheets — they make overriding painful.

**`:is()` vs `:where()`** — both match any selector in their list. `:where()` always has zero specificity — use it for grouping that can be overridden without fighting.

### Value Processing Pipeline

1. **Declared** — what you wrote
2. **Cascaded** — the winning declaration
3. **Specified** — after applying defaults
4. **Computed** — relative units resolved (`em` → `px`; percentages stay)
5. **Used** — after layout (percentages → pixels)
6. **Actual** — after constraints (subpixel rounding)

`getComputedStyle()` returns the **resolved value** — a mix of computed and used values depending on the property. `width: auto` returns the used pixel value. `transform` returns the computed matrix.

### Inheritance

Inheriting properties: `color`, `font-*`, `line-height`, `text-*`, `cursor`, `visibility`.
Non-inheriting: `margin`, `padding`, `border`, `background`, layout properties.

```css
.child { color: inherit; }   /* force inheritance */
.child { color: initial; }   /* reset to initial value */
.child { color: revert; }    /* what browser would use without author styles */
```

---

## 2. The Box Model

```css
/* Default — width applies to content only */
box-sizing: content-box;

/* Better — width includes padding and border */
*, *::before, *::after { box-sizing: border-box; }
```

Set `border-box` globally. Always.

### Margin Collapse

Vertical margins between block elements collapse — the larger applies, not both. Horizontal margins never collapse.

Collapse is prevented by: flex/grid container, `overflow` other than `visible`, padding or border between parent and first/last child, absolutely positioned elements.

---

## 3. Units — The Full System

### Relative Units

**`em`** — relative to the element's own `font-size`. Compounds when nested. Use for: properties that should scale with local text size (button padding, line-height as a multiplier).

**`rem`** — relative to root `<html>` font-size. Does not compound. Predictable. Use for: spacing, sizing — the correct default for most decisions.

**`ch`** — width of the `0` character. Use for: `max-width` on text columns. `60ch–75ch` is the typographically correct line length range.

**`lh`** — equal to the element's `line-height`. Use for spacing proportional to leading.

**`cap`** — height of capital letters. Use for icon sizing relative to text.

### Viewport Units

```css
vw   /* 1% of viewport width */
vh   /* 1% of viewport height — problematic on mobile */
svh  /* small viewport height — excludes browser chrome */
lvh  /* large viewport height — includes browser chrome space */
dvh  /* dynamic — updates as chrome shows/hides; causes reflows, use carefully */
vmin /* smaller of vw/vh */
vmax /* larger of vw/vh */
```

`vh` on mobile includes space behind the address bar when it's hidden — content sized to `100vh` overflows when the bar appears. Use `svh` for "exactly the visible area."

### Container Query Units

`cqi` — 1% of container inline size (width). `cqb` — 1% of container block size (height). Available after establishing a container context with `container-type`.

### `clamp()` — Fluid Scaling

```css
/* clamp(minimum, preferred, maximum) */
font-size: clamp(1rem, 2.5vw, 1.5rem);
padding:   clamp(1.5rem, 5vw, 4rem);
width:     clamp(280px, 50%, 640px);
```

`clamp()` eliminates breakpoints for typography and spacing. The preferred value can be any expression: `calc()`, viewport units, container units.

### `calc()` and Math Functions

```css
width: calc(100% - 2rem);
width: min(100%, 640px);      /* whichever is smaller */
font-size: max(1rem, 2vw);    /* whichever is larger */
width: round(var(--w), 1px);  /* snap to pixel grid */
```

---

## 4. Typography — Technical System

### Root Font Size — The Foundation

```css
html {
  font-size: 100%;  /* respect browser default — never set px here */
  /* or */
  font-size: clamp(100%, 1rem + 0.5vw, 120%); /* fluid root */
}
```

**Never set `font-size` in `px` on `html` or `body`.** It overrides user browser font size preferences — a direct accessibility violation.

### Type Scale

```css
:root {
  --ratio:   1.25;
  --fs-sm:   calc(var(--fs-base) / var(--ratio));
  --fs-base: 1rem;
  --fs-md:   calc(var(--fs-base) * var(--ratio));
  --fs-lg:   calc(var(--fs-md)   * var(--ratio));
  --fs-xl:   calc(var(--fs-lg)   * var(--ratio));
  --fs-2xl:  calc(var(--fs-xl)   * var(--ratio));
}
```

A modular scale produces sizes with consistent harmonic relationships. Ratios: minor third (1.2), major third (1.25), perfect fourth (1.333).

### @font-face — Correct Setup

```css
@font-face {
  font-family: 'Primary';
  src: url('/fonts/primary.woff2') format('woff2');
  font-weight: 300 900;    /* range for variable fonts */
  font-style: normal;
  font-display: swap;
  unicode-range: U+0000-00FF;
}
```

**`font-display` values:**
- `block` — invisible text briefly, then font
- `swap` — fallback immediately, swap when ready (risk of layout shift if metrics differ)
- `fallback` — 100ms block, 3s swap window, keep fallback after
- `optional` — 100ms block, no swap; best performance; use for body text

### Fallback Font Metrics — Eliminating CLS on Swap

```css
@font-face {
  font-family: 'Primary-fallback';
  src: local('Georgia');
  size-adjust: 106%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}

body { font-family: 'Primary', 'Primary-fallback', serif; }
```

Matching fallback metrics eliminates layout shift when the webfont swaps in.

### Variable Fonts

```css
@font-face {
  font-family: 'Variable';
  src: url('/fonts/variable.woff2') format('woff2-variations');
  font-weight: 100 900;
}

/* Standard properties — prefer over font-variation-settings */
h1 { font-weight: 650; } /* any value, not just multiples of 100 */

/* Low-level axis control */
h1 { font-variation-settings: 'wght' 650, 'wdth' 85, 'OPSZ' 24; }
```

Use standard CSS properties (`font-weight`, `font-stretch`) where possible — they inherit correctly. Use `font-variation-settings` for custom axes or non-standard values.

`font-optical-sizing: auto` — browser adjusts optical size axis based on `font-size`. On by default. Only disable when manually controlling `OPSZ`.

### Line Length, Wrapping, OpenType

```css
p, li { max-width: 65ch; } /* optimal reading line length */

h1, h2, h3 { text-wrap: balance; } /* distribute words evenly across lines */
p          { text-wrap: pretty;  } /* avoid orphans and awkward breaks */

body {
  font-kerning: normal;
  font-variant-ligatures: common-ligatures;
}

.figures { font-variant-numeric: oldstyle-nums proportional-nums; }
.caps    { font-variant-caps: small-caps; }
```

### Line Height

```css
/* Always unitless — multiplied by element's font-size, inherits correctly */
body { line-height: 1.6; }
h1   { line-height: 1.1; }

/* Never px — doesn't scale with font-size changes */
```

---

## 5. Layout — Grid and Flexbox at Depth

### Flexbox — One Axis

Flexbox is for one-dimensional layout: a row or a column. Navigation bars, button groups, centering, distributing space along one axis.

```css
.container {
  display: flex;
  flex-direction: row;           /* row | column | row-reverse | column-reverse */
  flex-wrap: wrap;
  gap: 1rem;
  justify-content: space-between; /* main axis */
  align-items: center;            /* cross axis */
  align-content: start;           /* cross axis when wrapping */
}

.item {
  flex: 1 1 200px;  /* grow shrink basis */
  flex: 1;          /* 1 1 0 — grow equally, basis 0 */
  flex: auto;       /* 1 1 auto — grow, basis = content size */
  flex: none;       /* 0 0 auto — rigid */

  align-self: flex-end;
  min-width: 0; /* IMPORTANT — default is auto, prevents shrinking below content size */
}
```

**`min-width: 0` on flex items** — flex items default to `min-width: auto`, meaning they won't shrink below their content size. This causes overflow. Override with `min-width: 0` when items need to shrink.

### Grid — Two Axes

Grid is for two-dimensional layout. Page structure, card grids, anything that needs row and column control simultaneously.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); /* responsive without media queries */
  grid-template-rows: auto 1fr auto;
  gap: 2rem 1rem; /* row-gap column-gap */
}

/* Place items explicitly */
.item {
  grid-column: 1 / 3;    /* line 1 to line 3 */
  grid-column: span 2;   /* span 2 tracks */
  grid-row: 1;
}

/* Named areas */
.layout {
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
```

### `auto-fill` vs `auto-fit`

```css
/* auto-fill — creates tracks even if empty; items don't stretch to fill */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));

/* auto-fit — collapses empty tracks; items stretch to fill available space */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
```

Use `auto-fit` when 2 items in a wide container should stretch and fill. Use `auto-fill` when empty columns should maintain structure.

### Subgrid

```css
.grid { display: grid; grid-template-columns: repeat(3, 1fr); }

.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* card rows participate in parent grid */
}
```

The correct solution for aligning internal elements across independently-sized cards. Well-supported in all modern browsers.

### Intrinsic Sizing

```css
width: min-content;  /* shrink to smallest without overflow */
width: max-content;  /* expand to largest without wrapping */
width: fit-content;  /* max-content capped at available space */

/* In grid tracks */
grid-template-columns: fit-content(300px) 1fr;
/* column is max-content but no wider than 300px */
```

---

## 6. Custom Properties — Design System Foundation

### Scope and Cascade

```css
:root {
  --color-text: #1a1a1a;
  --spacing-unit: 0.5rem;
  --radius: 4px;
}

/* Scoped — overrides :root values for this subtree */
.card { --color-text: white; }
```

Custom properties cascade and inherit exactly like standard properties.

### Spacing Scale

```css
:root {
  --space-unit: 8px;
  --space-1:  calc(var(--space-unit) * 0.5);   /* 4px  */
  --space-2:  var(--space-unit);               /* 8px  */
  --space-3:  calc(var(--space-unit) * 1.5);   /* 12px */
  --space-4:  calc(var(--space-unit) * 2);     /* 16px */
  --space-6:  calc(var(--space-unit) * 3);     /* 24px */
  --space-8:  calc(var(--space-unit) * 4);     /* 32px */
  --space-12: calc(var(--space-unit) * 6);     /* 48px */
}
```

Use tokens. `padding: var(--space-4)` is readable and consistent. `padding: 8px` is neither.

### Color — Modern Approach

```css
:root {
  /* oklch — perceptually uniform, wide gamut capable */
  --color-brand:    oklch(55% 0.18 255);
  --color-brand-lt: oklch(75% 0.12 255);

  /* Semantic tokens — what the color means, not what it is */
  --color-text:    oklch(15% 0 0);
  --color-surface: oklch(98% 0 0);
  --color-border:  oklch(85% 0 0);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-text:    oklch(92% 0 0);
    --color-surface: oklch(12% 0 0);
    --color-border:  oklch(25% 0 0);
  }
}
```

**`oklch(lightness chroma hue)`** — perceptually uniform. Changing lightness doesn't shift perceived hue. A palette with consistent lightness steps actually looks consistent. Prefer over `hsl`.

### `color-mix()` and Relative Color Syntax

```css
/* Mix two colors */
background: color-mix(in oklch, var(--color-brand) 30%, white);

/* Derive from existing — same hue/chroma, different lightness */
--color-brand-light: oklch(from var(--color-brand) calc(l + 20%) c h);

/* Same color, reduced opacity */
--color-brand-alpha: oklch(from var(--color-brand) l c h / 0.5);
```

Generate a full palette from a single brand color token — no preprocessors needed.

### Custom Properties as JS Bridge

```css
.parallax {
  transform: translateY(calc(var(--scroll-offset, 0) * 0.5px));
}
```

```js
document.documentElement.style.setProperty('--scroll-offset', window.scrollY);
```

Visual logic lives in CSS. JS supplies dynamic values. The fallback `var(--prop, fallback)` handles initial state before JS runs.

---

## 7. Responsive and Adaptive Design

### Media Queries — Mobile First

```css
/* Base — mobile */
.container { padding: var(--space-4); }

/* Larger viewports add to the base */
@media (min-width: 640px)  { .container { padding: var(--space-6); } }
@media (min-width: 1024px) { .container { max-width: 1200px; margin-inline: auto; } }
```

Mobile-first means base styles target the smallest viewport, enhanced for larger. Structurally correct — adding is more maintainable than removing.

### Media Features Worth Knowing

```css
/* Viewport */
@media (min-width: 768px) { }
@media (width >= 768px)   { } /* modern range syntax — equivalent */

/* User preferences */
@media (prefers-color-scheme: dark)          { }
@media (prefers-reduced-motion: reduce)      { }
@media (prefers-contrast: more)              { }
@media (prefers-reduced-transparency: reduce){ }
@media (forced-colors: active)               { } /* Windows High Contrast */

/* Device capability */
@media (pointer: coarse) { } /* touch — increase tap targets */
@media (pointer: fine)   { } /* mouse */
@media (hover: hover)    { } /* device supports hover */
@media (hover: none)     { } /* touch, no hover */
```

### Container Queries — Component-Level Response

```css
/* Establish container context */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* Query the container, not the viewport */
@container card (min-width: 400px) {
  .card { flex-direction: row; }
}

@container (min-width: 600px) {
  .card-title { font-size: var(--fs-lg); }
}
```

`container-type: inline-size` — enables width-based queries. `container-type: size` — both axes, but removes element from normal flow for sizing. Use `inline-size` unless height queries are specifically needed.

### Logical Properties — Writing Mode Agnostic

```css
/* Physical */
margin-left: auto;
padding-top: 1rem;
width: 100%;

/* Logical equivalents */
margin-inline-start: auto;  /* left in LTR, right in RTL */
padding-block-start: 1rem;  /* top in horizontal writing mode */
inline-size: 100%;           /* width in horizontal writing mode */
```

`inline` = axis text flows along. `block` = perpendicular. `start`/`end` = beginning and end of axis.

---

## 8. Modern CSS Features

### Cascade Layers — Architecture Without Specificity Wars

```css
@layer reset, base, components, utilities;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
}

@layer base {
  body { font-family: var(--font-body); color: var(--color-text); }
}

@layer components {
  .card { /* ... */ }
}

@layer utilities {
  .sr-only { /* ... */ }
}
```

Unlayered CSS beats all layered CSS regardless of specificity. Use layers for architecture; unlayered for overrides.

### `:has()` — Parent and Relational Selection

```css
/* Style form containing invalid input */
form:has(:invalid) { border-color: red; }

/* Style card containing a figure */
.card:has(figure) { padding-top: 0; }

/* Style label when its input is checked */
label:has(input:checked) { font-weight: 700; }
```

The parent selector the web needed for decades. Full support in all modern browsers.

### `@property` — Typed Custom Properties

```css
@property --hue {
  syntax: '<number>';
  inherits: false;
  initial-value: 220;
}

/* Now this transition actually works */
.swatch {
  background: oklch(60% 0.2 var(--hue));
  transition: --hue 0.3s ease;
}
.swatch:hover { --hue: 40; }
```

Without `@property`, custom properties are untyped strings — they cannot be transitioned or animated. With it, typed properties can be interpolated, enabling animation of values feeding into color functions, transforms, or any CSS expression.

### Scroll-Driven Animations

```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.section {
  animation: fade-in linear both;
  animation-timeline: view();    /* fires as element enters/exits viewport */
  animation-range: entry 0% entry 40%;
}

/* Page scroll progress indicator */
.progress-bar {
  transform-origin: left;
  animation: grow-width linear;
  animation-timeline: scroll(root block);
}
@keyframes grow-width {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}
```

`view()` — element-based, equivalent to IntersectionObserver. `scroll()` — scroll position of a container. Check Can I Use — Chrome/Edge full, Firefox/Safari improving.

---

## 9. Selectors — Reference

```css
/* Attribute */
[href^="https"]  /* starts with */
[href$=".pdf"]   /* ends with */
[class*="icon-"] /* contains */

/* Structural */
:nth-child(2n+1)    /* odd */
:nth-child(3n)      /* every third */
:nth-child(-n+3)    /* first three */
:nth-last-child(2)  /* second from end */

/* Relational */
:is(h1, h2, h3)       /* match any; takes specificity of most specific */
:where(.class)        /* zero-specificity grouping */
:not(.excluded)       /* negation; takes argument's specificity */
:has(> .direct-child) /* has a direct child matching */

/* State */
:focus-visible    /* focused via keyboard, not mouse click */
:focus-within     /* element or any descendant is focused */
:empty            /* no children including text nodes */
:checked          /* checkbox/radio/option */
:placeholder-shown
:disabled / :enabled
:valid / :invalid
:in-range / :out-of-range

/* Pseudo-elements */
::before / ::after
::first-line / ::first-letter
::selection       /* user-selected text */
::placeholder
::marker          /* list item bullet or number */
::backdrop        /* behind a dialog or fullscreen element */
```

---

## 10. Animation and Transition

### Transitions

```css
.button {
  transition:
    background-color 200ms ease,
    transform 150ms cubic-bezier(0.34, 1.56, 0.64, 1), /* overshoot/spring */
    box-shadow 200ms ease;
}

/* Never transition `all` — catches unexpected properties, hurts performance */

/* GPU-composited only — best performance */
.panel { transition: transform 300ms ease, opacity 300ms ease; }
```

### @keyframes

```css
@keyframes slide-up {
  from { transform: translateY(100%); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}

.modal {
  animation:
    slide-up
    400ms
    cubic-bezier(0.16, 1, 0.3, 1)
    100ms          /* delay */
    1              /* iterations */
    normal         /* direction */
    forwards;      /* fill — retain end state */
}
```

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  /* Make instantaneous, don't remove — abrupt changes are less jarring than ignoring the preference */
  .animated {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 11. Accessibility via CSS

### Focus Management

```css
/* Visible for keyboard, invisible for mouse click */
:focus-visible {
  outline: 2px solid var(--color-focus);
  outline-offset: 3px;
}

/* Never: */
:focus { outline: none; } /* removes all focus indicators */
```

`:focus-visible` is the correct selector for custom focus styles. Do not remove focus styles without replacement.

### Visually Hidden

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Reveal when focused — for skip links */
.sr-only:focus {
  position: static;
  width: auto;
  height: auto;
  margin: 0;
  overflow: visible;
  clip: auto;
  white-space: normal;
}
```

### Touch Targets

```css
/* Minimum 44×44px — Apple HIG and WCAG 2.5.5 */
.button { min-height: 44px; padding: 12px 16px; }

/* Expand hit area without changing visual size */
.icon-button { position: relative; }
.icon-button::after {
  content: '';
  position: absolute;
  inset: -8px;
}
```

---

## 12. Stacking and Positioning

### `position` Values

```css
static;   /* default. Not positioned. z-index has no effect. */
relative; /* offset from normal position. Creates stacking context if z-index set. */
absolute; /* removed from flow. Relative to nearest positioned ancestor. */
fixed;    /* relative to viewport. Broken inside transformed ancestors. */
sticky;   /* relative until scroll threshold, then fixed within its container. */
```

`position: fixed` is broken inside a parent with `transform`, `filter`, `perspective`, or `will-change: transform`. It positions relative to that ancestor, not the viewport. Intentional browser behaviour.

### Stacking Context

Z-index only has meaning between elements in the same stacking context.

**What creates a stacking context:**
- `position: relative/absolute/fixed/sticky` + non-auto `z-index`
- `opacity` < 1
- `transform` other than `none`
- `filter` other than `none`
- `will-change` targeting any of the above
- `isolation: isolate` — creates context with no visual side effect

**`isolation: isolate`** is the correct tool when you need a stacking context without visual changes. Use on components that must contain their own z-index stack.

### `inset` Shorthand

```css
position: absolute;
inset: 0;            /* top: 0; right: 0; bottom: 0; left: 0 */
inset-block: 0;      /* top and bottom */
inset-inline: 0;     /* left and right */
```

---

## 13. Performance

- **`content-visibility: auto`** — skips rendering off-screen content. Major gain for long pages.
  ```css
  .section { content-visibility: auto; contain-intrinsic-size: 0 500px; }
  ```
- **Never animate layout properties** — `width`, `height`, `margin` trigger Layout and Paint every frame
- **`will-change` sparingly** — each promoted layer costs GPU memory; use only before known animations
- **No `transition: all`** — catches unexpected properties, increases paint work
- **No CSS `@import`** — blocks rendering; use `<link>` tags or bundling instead

---

## 14. Canonical References

- **MDN CSS Reference** — `developer.mozilla.org/en-US/docs/Web/CSS/Reference` — every property, value, selector with examples and browser support
- **CSS Specifications** — `drafts.csswg.org` — the actual specs; go here when MDN leaves ambiguity
- **Can I Use** — `caniuse.com` — check browser support before using any modern feature
- **web.dev/learn/css** — Google's structured CSS course; reliable and current
- **oklch.com** — interactive oklch color picker; essential for working in oklch color space
- **CSS Tricks Almanac** — `css-tricks.com/almanac` — well-maintained property and selector reference
