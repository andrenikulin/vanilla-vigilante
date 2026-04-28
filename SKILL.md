---
name: vanilla-vigilante
description: >
  Expert-level knowledge of the HTML, CSS, and JavaScript trinity for building static websites — handwritten, unabstracted, close to the metal. Use this skill whenever the task involves any part of a static site: semantic markup, document architecture, CSS systems, layout, typography, units, custom properties, JavaScript behavior, DOM manipulation, animation, observer APIs, event handling, performance, accessibility, responsive design, build tooling decisions, or code critique. Triggers even when the user doesn't say "vanilla" — if they're building or reviewing a website without a framework, this skill applies. Also triggers explicitly for evaluation and critique tasks: when existing HTML, CSS, or JS needs to be assessed for quality, correctness, or whether the approach should be reconsidered.
---

# Vanilla Vigilante

A static website is not a lesser product. It is a precise one.

Vanilla means handwritten — HTML that carries meaning, CSS that encodes a system, JavaScript that works with the browser rather than around it. No framework manages these for you. That is not a limitation. It is the condition under which craft becomes visible.

---

## Core Philosophy

**The browser is already a powerful platform.** Most of what frameworks provide exists natively. The skill is knowing where to find it, how it behaves, and when the native version is sufficient.

**Complexity must earn its place.** Every abstraction, every build step, every dependency has a cost. That cost must be justified by a concrete benefit to this specific project.

**Static doesn't mean simple.** Static means no server, no runtime data fetching, no backend. It says nothing about visual sophistication, interactivity, or animation depth.

**The trinity is one material.** HTML, CSS, and JavaScript are not separate concerns. CSS reads HTML structure. JS reads and writes both. Design decisions propagate across all three. Think in all three simultaneously.

---

## Reference Files

Read the relevant file before writing or reviewing. For most tasks, read all three — they are dense, not long, and they inform each other.

### `references/html.md`
The anchor. Document architecture, semantic element meaning, the correct `<head>`, resource hints, images (`alt`, `width`/`height`, LCP treatment, `<picture>`, `srcset`/`sizes`), links vs buttons, forms with full accessible markup, ARIA when HTML isn't enough, interactive patterns (`<dialog>`, disclosure, skip links), global attributes including `inert`, structured data, script loading strategy.

**Read for:** any markup task, accessibility questions, document structure, form building, image implementation, metadata.

### `references/css.md`
The system. Cascade and specificity at real depth, value resolution pipeline, the box model, the full unit system (`rem`, `em`, `ch`, viewport units, `clamp()`), typography as a technical system (root font size, type scale, `@font-face`, variable fonts, fallback metrics, `text-wrap`), layout (Grid and Flexbox at depth, subgrid, intrinsic sizing), custom properties as a design system (spacing scale, color in `oklch`, the JS bridge pattern), responsive and adaptive design (media queries, container queries, logical properties), modern CSS (cascade layers, `:has()`, `@property`, scroll-driven animations, `color-mix()`, relative color), selectors, animation and transition, accessibility via CSS (`:focus-visible`, `.sr-only`, touch targets), stacking contexts, `isolation: isolate`, performance.

**Read for:** any CSS task, layout decisions, typography, color systems, animation, responsive design, modern CSS feature usage.

### `references/js-dom.md`
The engine. Browser rendering pipeline at real depth (style → layout → paint → composite, compositor layers, `will-change`, layer explosion), layout thrash — what causes it and how to prevent it, the JavaScript engine (hidden classes, inline caches, deoptimisation, GC pressure in animation loops), the event loop (microtask vs macrotask vs rendering, precise order, `queueMicrotask`, `requestIdleCallback`), DOM selection (live vs static collections), manipulation and what each operation costs, geometry APIs (`getBoundingClientRect` vs `offsetTop` — the real difference), events in full (propagation, delegation, custom events, `AbortController` for cleanup, pointer events, `passive: true`), observer APIs (IntersectionObserver with threshold and rootMargin precision, ResizeObserver with `devicePixelContentBoxSize` for canvas, MutationObserver), animation (frame-rate-independent `requestAnimationFrame`, Web Animations API with full control, scroll-linked patterns, reduced motion), async patterns for static sites, module architecture (ES modules, private class fields, state with plain objects), Web APIs (History, `matchMedia`, canvas DPR setup, CSS custom properties from JS), performance (DevTools — what to actually look at, memory leak patterns, script loading), security.

**Read for:** any JavaScript task, DOM work, animation, performance investigation, event handling, canvas, browser API usage.

---

## Build Tooling — Decision Reference

**No build step** — valid for simple projects. Native ES modules, single CSS file, minimal JS. Zero configuration. Appropriate when: one or two JS files, no asset processing needed.

**esbuild** — the correct default when a build step is warranted. Sub-millisecond builds, ES module bundling, tree-shaking, CSS processing, asset handling. Reaches for this before anything else.

**Vite** — acceptable if a development server with HMR improves the workflow. Adds more opinion and dependencies than esbuild alone.

**Webpack / Parcel / others** — not worth the complexity for a static site without a specific justification.

A build step earns its cost when: you have enough modules that multiple HTTP requests are a measurable performance problem; you need to process or optimise assets alongside JS; you want TypeScript or modern syntax with predictable output.

---

## Critique Layer — The Capricorn Function

When the task is to evaluate existing code, apply this without softening the findings. The goal is an honest assessment of what works, what doesn't, and what should be reconsidered.

### HTML Critique
- Is the document structure semantic, or is it `<div>` all the way down?
- Are headings in logical order, or chosen for visual size?
- Do images have `alt`, `width`, and `height`? Is the LCP image lazy-loaded (it shouldn't be)?
- Are interactive elements `<button>` and `<a>`, or `<div>` with click handlers?
- Is the `<head>` complete — charset, viewport, title, preloads in the right order?
- Are forms using `<label>`, `autocomplete`, `inputmode`, and `aria-describedby` correctly?

### CSS Critique
- Is sizing in `px` where `rem` or `clamp()` should be — overriding user font preferences?
- Is the specificity architecture a mess — IDs, `!important`, redundant overrides?
- Are animations on layout-triggering properties (`width`, `top`) instead of `transform`/`opacity`?
- Is there a type scale and spacing system, or magic numbers everywhere?
- Is `transition: all` used — catching unexpected properties and hurting performance?
- Are responsive patterns using viewport-first (desktop) instead of mobile-first?

### JavaScript Critique
- Is there layout thrash — reads and writes interleaved in loops?
- Are scroll or resize listeners running raw without throttling via `rAF`?
- Are event listeners cleaned up, or accumulating as memory leaks?
- Is `innerHTML` used where it destroys listeners or opens XSS risk?
- Are animations running inside `requestAnimationFrame` with frame-rate independence?
- Is state scattered, or is there a clear source of truth?
- Are `var` or poorly scoped variables creating unpredictable behaviour?

### Cross-Cutting
- Is something done in JS that CSS now handles natively?
- Is the complexity of this code proportionate to what it actually does?
- Are there dependencies that add weight without adding capability?

### When to Stop Polishing

Code is done when: it does what it needs to do, it doesn't introduce performance problems, it's readable six months from now, and it doesn't make the next change harder. Polishing code no user encounters is a misallocation. If the bottleneck is visual, fix it visually. If the bottleneck is in logic no one will see, leave it.

### When to Reconsider the Approach

If managing a UI behaviour requires significantly more code than a native CSS feature would, the answer is CSS — not better JS. If state management has grown into a substantial part of what should be a static site's codebase, the architecture needs to be questioned before it grows further. If a pattern requires workarounds for its own workarounds, back up and find the simpler path.

### When Vanilla Is the Wrong Answer

Vanilla is wrong when: the site requires real-time data from a backend with complex update patterns; the team is large enough that a shared component model prevents coordination failures; the project needs server-side rendering for SEO on dynamic content. These are not the conditions of a portfolio site, a hospitality brand, or a case study collection. Be honest about which situation you're actually in.
