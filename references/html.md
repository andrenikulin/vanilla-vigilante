# HTML — Deep Reference

HTML is not a formatting language. It is a meaning language. Every element is a declaration about what content is — not how it looks. When the structure is right, CSS has something real to attach to, JavaScript has something stable to traverse, browsers know how to present content to assistive technologies, and search engines understand the document. When structure is wrong, everything built on top of it is compensating for the foundation.

---

## 1. The Document — Architecture Before Content

### The `<head>` — Complete and Correct

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Character encoding — must be first, within first 1024 bytes -->
  <meta charset="UTF-8">

  <!-- Viewport — mandatory for responsive design -->
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <!-- Title — tab, bookmarks, search results. 50–60 characters optimal. -->
  <title>Page Title — Site Name</title>

  <!-- Description — search snippet. 150–160 characters. High effect on click-through. -->
  <meta name="description" content="...">

  <!-- Canonical — preferred URL for this content -->
  <link rel="canonical" href="https://example.com/page">

  <!-- Theme color — browser chrome color on mobile -->
  <meta name="theme-color" content="#1a1a2e">
  <meta name="theme-color" content="#f5f5f0" media="(prefers-color-scheme: light)">

  <!-- Favicon — modern approach -->
  <link rel="icon" href="/favicon.svg" type="image/svg+xml">
  <link rel="icon" href="/favicon.ico" sizes="32x32">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png"> <!-- 180×180 -->

  <!-- Open Graph -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="...">
  <meta property="og:image" content="https://example.com/og-image.jpg"> <!-- 1200×630 -->
  <meta property="og:url" content="https://example.com/page">
  <meta property="og:type" content="website">

  <!-- Preload critical assets — before stylesheets -->
  <link rel="preload" href="/fonts/primary.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/images/hero.avif" as="image" fetchpriority="high">

  <!-- Stylesheets -->
  <link rel="stylesheet" href="/css/main.css">

  <!-- Scripts — deferred by default for your own code -->
  <script defer src="/js/main.js"></script>
  <script type="module" src="/js/app.js"></script>
</head>
```

### `lang` Attribute — Not Optional

```html
<html lang="en">
<html lang="fr">
<html lang="ru">
<html lang="zh-Hans"> <!-- Simplified Chinese -->
<html lang="pt-BR">   <!-- Brazilian Portuguese -->
```

`lang` tells screen readers which language engine to use for pronunciation. Missing `lang` means a screen reader may read French text with English phonemes. It also affects browser spell-check, `quotes` property defaults in CSS, and text-to-speech. Set it on `<html>` always. Set it on inline elements when content switches language.

### Viewport — What the Values Actually Mean

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

- `width=device-width` — sets viewport to physical device width, not the mobile default of 980px
- `initial-scale=1` — no initial zoom

Never add `user-scalable=no` or `maximum-scale=1`. These prevent users from zooming — a direct WCAG 1.4.4 failure that harms users with low vision.

---

## 2. Semantic Elements — What They Actually Mean

Semantic elements communicate document structure to browsers, assistive technologies, and search engines. Choose the element that describes what content *is*, not what it looks like.

### Document Structure

```html
<header>
  <!-- Introductory content for its nearest sectioning ancestor.
       Can appear as the page header, or inside article/section. -->

<nav>
  <!-- A group of navigation links. Not every link group —
       only major navigation. Multiple <nav> elements are valid
       (site nav, breadcrumb, table of contents, pagination). -->

<main>
  <!-- The dominant content. One per page.
       Must not be inside article, aside, footer, header, or nav. -->

<article>
  <!-- Self-contained content that could be distributed independently.
       A case study, a blog post, a card, a comment. -->

<section>
  <!-- Thematic grouping, typically with a heading.
       Not a generic wrapper — use <div> for that. -->

<aside>
  <!-- Tangentially related content.
       Sidebars, pull quotes, supplementary material. -->

<footer>
  <!-- Footer for its parent sectioning element.
       Authorship, copyright, related links, contact. -->
```

**`<section>` requires a heading to be meaningful.** If a `<section>` has no heading, it is probably a `<div>`. If a `<div>` has a heading and groups related content, it is probably a `<section>`.

**`<article>` can nest.** A list of articles can be wrapped in an outer `<article>`. Each should make sense independently.

### Headings — Hierarchy, Not Size

```html
<h1>Primary subject of the page</h1>
  <h2>Major section</h2>
    <h3>Subsection</h3>
```

Headings create the document outline. Screen readers let users navigate by heading — `h1` → `h2` → `h3` — to understand structure and jump to sections.

Rules: one `<h1>` per page. Never skip levels. Never choose heading level for its visual size — CSS controls size, HTML controls meaning. Every `<section>` should have a heading at the appropriate level.

### Text-Level Semantics

```html
<strong>   <!-- Strong importance, seriousness, urgency. Semantic bold. -->
<em>       <!-- Stress emphasis — changes meaning. Not decorative italic. -->
<b>        <!-- Attention without importance: keywords, product names. No semantic weight. -->
<i>        <!-- Alternate voice: foreign words, technical terms, thoughts. No semantic weight. -->
<mark>     <!-- Highlighted/relevant text in current context. Not decoration. -->
<small>    <!-- Fine print, legal text, secondary information. -->
<s>        <!-- No longer accurate or relevant. Not for deletion (use <del>). -->
<del datetime="2024-01-01"> <!-- Deleted text. -->
<ins datetime="2024-01-15"> <!-- Inserted text. -->
<cite>     <!-- Title of a creative work. NOT the author's name. -->
<q>        <!-- Inline quotation. Browser adds locale-appropriate quote marks. -->
<blockquote cite="url">     <!-- Extended quotation. cite is the source URL. -->
<abbr title="HyperText Markup Language">HTML</abbr>
<time datetime="2024-03-15">March 15, 2024</time>
<address>  <!-- Contact info for nearest article or body ancestor. -->
<code>     <!-- Inline code. -->
<pre><code>...</code></pre> <!-- Code block. Whitespace preserved. -->
<kbd>      <!-- Keyboard input. -->
<samp>     <!-- Sample output from a program. -->
<var>      <!-- Variable in math or programming context. -->
<sub>      <!-- Subscript. Chemical formulas, footnotes. -->
<sup>      <!-- Superscript. Exponents, footnote references. -->
<wbr>      <!-- Word break opportunity hint for long unbreakable strings. -->
```

Screen readers announce `<strong>` and `<em>` with stress. They do not announce `<b>` and `<i>`. Choose based on whether the emphasis carries meaning.

### `<time>` — Machine-Readable Dates

```html
<time datetime="2024-03-15">March 15, 2024</time>
<time datetime="2024-03-15T14:30:00+03:00">2pm Moscow time</time>
<time datetime="PT2H30M">2 hours 30 minutes</time>
<time datetime="2024-W12">Week 12 of 2024</time>
```

`datetime` is machine-readable; element content is human-readable. Search engines extract structured date information from `datetime`.

---

## 3. Lists — Choosing the Right One

```html
<!-- Unordered — items with no meaningful sequence -->
<ul>
  <li>Navigation link</li>
  <li>Navigation link</li>
</ul>

<!-- Ordered — sequence matters -->
<ol>
  <li>First step</li>
  <li>Second step</li>
</ol>
<ol start="3" reversed>  <!-- start at 3; count down -->

<!-- Description list — name-value pairs -->
<dl>
  <dt>Client</dt>
  <dd>Fond Bank</dd>
  <dt>Year</dt>
  <dd>2023</dd>
  <dt>Role</dt>
  <dd>Visual Identity</dd>
</dl>
```

`<dl>` is the correct element for: glossaries, project metadata pairs, FAQs (question/answer), any key-value data. Not just definitions.

Navigation menus are `<ul>` — links in no required order. Breadcrumbs and process steps are `<ol>` — sequence is meaningful.

---

## 4. Links and Buttons — The Correct Element

The most common semantic mistake on the web: using `<div>` or `<span>` for interactive elements.

```html
<!-- LINK — navigates to a URL or anchor -->
<a href="/work/fond-bank">View case study</a>

<!-- BUTTON — triggers an action, no navigation -->
<button type="button">Open menu</button>
<button type="submit">Send message</button>
<button type="reset">Clear form</button>
```

**If it goes somewhere, it's a link. If it does something, it's a button.**

A `<div>` with a click handler is never the correct choice. It has no keyboard access, no screen reader role, no built-in focus management. If you find yourself writing `role="button"` on a `<div>`, use `<button>`.

Always set `type` on `<button>`. The default is `submit` — a button inside a form without an explicit type will submit the form.

### Link Attributes

```html
<!-- New tab — always warn the user -->
<a href="..." target="_blank" rel="noopener noreferrer">
  External link <span class="sr-only">(opens in new tab)</span>
</a>
<!-- rel="noopener" — prevents new tab accessing window.opener (security)
     rel="noreferrer" — also suppresses Referer header -->

<!-- Download -->
<a href="/downloads/brief.pdf" download="project-brief.pdf">Download brief</a>

<!-- Email and phone -->
<a href="mailto:hello@example.com">hello@example.com</a>
<a href="tel:+74951234567">+7 495 123-45-67</a>

<!-- In-page anchor -->
<a href="#contact">Jump to contact</a>
<section id="contact">...</section>
```

---

## 5. Images — Complete

### Every Attribute That Matters

```html
<img
  src="/images/cover.jpg"
  alt="Geometric polygon construction lines forming the Fond Bank grid system"
  width="1200"
  height="800"
  loading="lazy"
  decoding="async"
  fetchpriority="low"
>
```

**`alt`** — a replacement for the image, not a description of it. What would a sighted user gain from this image that a screen reader user would miss? Decorative images: `alt=""` (empty, not omitted). Functional images (icon-only button): describe the function, not the appearance.

**`width` and `height`** — always set both. The browser uses them to reserve space before the image loads, preventing CLS. Set to the image's intrinsic dimensions — CSS controls display size.

**`loading="lazy"`** — defer loading until near the viewport. Use on all images except above-the-fold.

**`decoding="async"`** — decode off the main thread. Reduces jank on image-heavy pages.

**`fetchpriority="high"`** — for the LCP candidate. `"low"` for below-fold images. `"auto"` (default) otherwise.

### The LCP Image — Special Treatment

```html
<!-- Hero or first significant image — never lazy-load this -->
<img
  src="/images/hero.avif"
  alt="..."
  width="1600"
  height="900"
  fetchpriority="high"
  decoding="sync"
>
```

Combined with preload in `<head>`:
```html
<link rel="preload" as="image" href="/images/hero.avif" fetchpriority="high">
```

### `<picture>` — Format Switching and Art Direction

```html
<!-- Format selection — browser picks first supported format -->
<picture>
  <source srcset="/images/photo.avif" type="image/avif">
  <source srcset="/images/photo.webp" type="image/webp">
  <img src="/images/photo.jpg" alt="..." width="800" height="600" loading="lazy">
</picture>

<!-- Art direction — different crop at different viewport sizes -->
<picture>
  <source media="(min-width: 800px)" srcset="/images/photo-wide.avif" type="image/avif">
  <source media="(min-width: 800px)" srcset="/images/photo-wide.webp" type="image/webp">
  <img src="/images/photo-portrait.jpg" alt="..." width="800" height="1000" loading="lazy">
</picture>
```

The `<img>` is what gets rendered — `<source>` elements provide alternatives. The `<img>` fallback is mandatory. Image format priority: AVIF → WebP → JPEG/PNG.

### `srcset` and `sizes` — Responsive Images

```html
<img
  src="/images/photo-800.jpg"
  srcset="
    /images/photo-400.jpg   400w,
    /images/photo-800.jpg   800w,
    /images/photo-1600.jpg 1600w
  "
  sizes="
    (min-width: 1024px) 800px,
    (min-width: 640px)  calc(100vw - 4rem),
    100vw
  "
  alt="..."
  width="800"
  height="600"
  loading="lazy"
>
```

`srcset` — available files and their widths. `sizes` — how wide the image will display at each viewport. The browser combines both to decide which file to fetch. `sizes` is evaluated before CSS so it must match what CSS will actually render.

### SVG — Inline vs External

```html
<!-- External — cached, simple, cannot be targeted by CSS -->
<img src="/icons/arrow.svg" alt="" width="24" height="24">

<!-- Inline — fully styleable, not separately cached -->
<svg width="24" height="24" viewBox="0 0 24 24" aria-hidden="true" focusable="false">
  <path d="M5 12h14M12 5l7 7-7 7"/>
</svg>

<!-- Icon button — label on the button, SVG hidden from AT -->
<button type="button" aria-label="Close panel">
  <svg width="24" height="24" viewBox="0 0 24 24" aria-hidden="true" focusable="false">
    <path d="M18 6L6 18M6 6l12 12"/>
  </svg>
</button>
```

`aria-hidden="true"` removes the SVG from the accessibility tree. `focusable="false"` prevents SVG focus in older browsers.

---

## 6. Resource Hints — Telling the Browser What's Coming

```html
<!-- preload — fetch now, high priority, critical current-page assets -->
<link rel="preload" as="font"   href="/fonts/primary.woff2" type="font/woff2" crossorigin>
<link rel="preload" as="image"  href="/images/hero.avif" fetchpriority="high">
<link rel="preload" as="style"  href="/css/critical.css">
<link rel="preload" as="script" href="/js/main.js">

<!-- prefetch — fetch for likely future navigation, low priority -->
<link rel="prefetch" href="/work/fond-bank">

<!-- preconnect — DNS + TCP + TLS handshake early -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- dns-prefetch — DNS only; fallback for browsers without preconnect -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com">
```

**Preload rules:**
- Only preload resources used on the current page
- Always set `as` — determines request priority and headers
- Font preloads require `crossorigin` even for same-origin fonts — fonts are always CORS
- Place preload links before stylesheets in `<head>` — they need to be discovered first

---

## 7. Tables — Correct Markup

Tables are for tabular data. Never for layout.

```html
<table>
  <caption>Project timeline — Fond Bank identity system</caption>

  <thead>
    <tr>
      <th scope="col">Phase</th>
      <th scope="col">Duration</th>
      <th scope="col">Deliverable</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <th scope="row">Discovery</th>
      <td>2 weeks</td>
      <td>Brief and audit</td>
    </tr>
    <tr>
      <th scope="row">Design</th>
      <td>6 weeks</td>
      <td>Identity system</td>
    </tr>
  </tbody>

  <tfoot>
    <tr>
      <td colspan="3">Timelines are approximate</td>
    </tr>
  </tfoot>
</table>
```

`<caption>` — the table's title, announced before the table content. Use instead of `aria-label` on `<table>`.

`scope="col"` / `scope="row"` — tells screen readers which cells the header applies to. Required for accessible tables. For complex multi-level headers, use `id` and `headers` attributes.

---

## 8. Forms — Full Semantic Markup

### Structure

```html
<form method="post" action="/contact" novalidate>
  <fieldset>
    <legend>Contact information</legend>

    <div class="field">
      <label for="name">Full name</label>
      <input
        id="name"
        name="name"
        type="text"
        autocomplete="name"
        required
        aria-required="true"
        aria-describedby="name-hint name-error"
      >
      <p id="name-hint" class="hint">As it appears on your ID</p>
      <p id="name-error" class="error" role="alert" hidden>Name is required</p>
    </div>

  </fieldset>
  <button type="submit">Send message</button>
</form>
```

`novalidate` — disables native browser validation UI so you can implement custom validation with full control. Still use HTML validation attributes (`required`, `type`, `pattern`) — they provide semantic meaning and a fallback.

### `<label>` — Always Explicit

```html
<!-- Explicit — preferred -->
<label for="email">Email address</label>
<input id="email" type="email">
```

Placeholder text is not a label. It disappears on focus, has low contrast by default, and is not reliably announced by screen readers. It is a hint. Always provide a visible `<label>`.

### Input Types — Use the Right One

```html
<input type="text">       <!-- generic single-line text -->
<input type="email">      <!-- validates email format; email keyboard on mobile -->
<input type="tel">        <!-- numeric keyboard on mobile; no format validation -->
<input type="url">        <!-- validates URL format; keyboard with .com key -->
<input type="number">     <!-- numeric with stepper; avoid for phone/PIN/credit card -->
<input type="search">     <!-- may show browser clear button -->
<input type="password">   <!-- masked text -->
<input type="date">       <!-- date picker; format varies by locale -->
<input type="checkbox">   <!-- boolean; one of many independent options -->
<input type="radio">      <!-- exclusive; one of a group -->
<input type="file">       <!-- file upload; use accept to filter types -->
<input type="hidden">     <!-- not rendered; submitted with form -->
```

`type="number"` has significant UX problems: stepper arrows, rejects leading zeros, locale-dependent decimal separator. For phone numbers, PINs, OTPs — use `type="text" inputmode="numeric"`.

### `inputmode` — Keyboard Without Type Constraints

```html
<input type="text" inputmode="numeric"  pattern="[0-9]*"> <!-- PIN, OTP -->
<input type="text" inputmode="decimal">  <!-- prices, measurements -->
<input type="text" inputmode="tel">      <!-- phone keyboard -->
<input type="text" inputmode="email">    <!-- email keyboard -->
<input type="text" inputmode="url">      <!-- URL keyboard -->
<input type="text" inputmode="none">     <!-- suppress keyboard; custom input provided -->
```

`inputmode` controls which virtual keyboard appears without changing the input's type validation.

### `autocomplete` — Correct Values

```html
<input autocomplete="name">
<input autocomplete="given-name">
<input autocomplete="family-name">
<input autocomplete="email">
<input autocomplete="tel">
<input autocomplete="street-address">
<input autocomplete="postal-code">
<input autocomplete="country-name">
<input autocomplete="new-password">     <!-- don't autofill existing password -->
<input autocomplete="current-password">
<input autocomplete="one-time-code">    <!-- OTP; triggers SMS autofill on iOS -->
<input autocomplete="cc-number">
<input autocomplete="cc-exp">
<input autocomplete="cc-csc">
```

Correct `autocomplete` values enable browser autofill and password managers. Required for WCAG 1.3.5.

### `aria-describedby` — Help Text and Errors

```html
<input
  id="password"
  type="password"
  aria-describedby="pw-requirements pw-error"
>
<p id="pw-requirements">Minimum 8 characters, one uppercase, one number</p>
<p id="pw-error" role="alert" hidden>Password does not meet requirements</p>
```

`aria-describedby` takes space-separated IDs. Screen readers announce the label, then value, then descriptions. Use for hints and error messages.

`role="alert"` — when content is added to this element, screen readers announce it immediately without waiting for user navigation.

---

## 9. ARIA — When and What

First rule: **don't use ARIA if native HTML provides the semantics.**

```html
<!-- BAD — needs extra keyboard handling, still inferior -->
<div role="button" tabindex="0">Click me</div>

<!-- GOOD — native element provides everything -->
<button type="button">Click me</button>
```

### Roles Worth Knowing

```html
role="alert"       <!-- urgent message; announced immediately on content change -->
role="status"      <!-- non-urgent; announced when AT is idle -->
role="dialog"      <!-- modal; requires aria-labelledby and focus management -->
role="tab"         <!-- tab in a tab panel -->
role="tablist"     <!-- container for tabs -->
role="tabpanel"    <!-- panel for a tab -->
role="switch"      <!-- on/off toggle -->
role="progressbar" <!-- use with aria-valuenow/min/max -->
role="tooltip"     <!-- shown on hover/focus -->
role="none"        <!-- removes from accessibility tree (synonym: presentation) -->
```

### Properties and States

```html
<!-- Labelling -->
aria-label="Close panel"               <!-- inline label; use when no visible text -->
aria-labelledby="heading-id extra-id"  <!-- points to visible text -->
aria-describedby="hint-id error-id"    <!-- supplementary description -->

<!-- States -->
aria-expanded="true|false"   <!-- disclosure/accordion button -->
aria-selected="true|false"   <!-- tab, option, treeitem -->
aria-checked="true|false|mixed" <!-- custom checkbox -->
aria-pressed="true|false|mixed" <!-- toggle button -->
aria-disabled="true"         <!-- logically disabled; still in AT, still focusable -->
aria-hidden="true"           <!-- remove from accessibility tree -->
aria-invalid="true"          <!-- invalid input; pair with error message -->
aria-required="true"         <!-- required field -->
aria-current="page"          <!-- current page in navigation -->
aria-current="step"          <!-- current step in a process -->

<!-- Live regions -->
aria-live="polite"     <!-- announces changes after current speech -->
aria-live="assertive"  <!-- interrupts immediately; use sparingly -->
aria-atomic="true"     <!-- announce entire region on any change -->

<!-- Relationships -->
aria-controls="panel-id"     <!-- this element controls another -->
aria-haspopup="menu|listbox|dialog" <!-- triggers a popup -->
```

### Live Regions — Correct Pattern

```html
<!-- Polite — status messages, non-urgent updates -->
<div aria-live="polite" aria-atomic="true" class="sr-only" id="status"></div>

<!-- Assertive — errors, critical time-sensitive messages -->
<div role="alert" id="error-message"></div>
```

The live region must exist in the DOM before content is added. Adding a live region and immediately inserting content is not reliably announced by all screen readers.

---

## 10. Interactive Patterns — Built Correctly

### Disclosure / Accordion

```html
<button type="button" aria-expanded="false" aria-controls="panel-1">
  About the project
</button>
<div id="panel-1" hidden>
  <p>Panel content</p>
</div>
```

```js
button.addEventListener('click', () => {
  const isExpanded = button.getAttribute('aria-expanded') === 'true';
  button.setAttribute('aria-expanded', String(!isExpanded));
  panel.hidden = isExpanded;
});
```

### Navigation with Current Page

```html
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/" aria-current="page">Home</a></li>
    <li><a href="/work">Work</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
```

### Skip Link

```html
<!-- First focusable element in the document -->
<a href="#main-content" class="sr-only">Skip to main content</a>

<main id="main-content" tabindex="-1">
  <!-- tabindex="-1" allows focus via JS without adding to tab order -->
</main>
```

Required for WCAG 2.4.1. Visible when focused, hidden otherwise.

### Native `<dialog>`

```html
<dialog id="project-modal" aria-labelledby="modal-title" aria-modal="true">
  <h2 id="modal-title">Fond Bank — Case Study</h2>
  <div class="dialog-content">...</div>
  <button type="button" autofocus>Close</button>
</dialog>
```

```js
const dialog = document.getElementById('project-modal');

dialog.showModal();  // traps focus, adds to top layer, adds ::backdrop

dialog.close();

// Backdrop click to dismiss
dialog.addEventListener('click', (e) => {
  if (e.target === dialog) dialog.close();
});
```

`showModal()` traps focus inside the dialog, adds it to the browser top layer (above everything including `position: fixed`), provides `::backdrop`, and handles Escape key automatically.

`autofocus` on the first interactive element — focus moves there when the dialog opens.

---

## 11. Global Attributes — The Complete Set Worth Knowing

```html
id="..."              <!-- unique identifier per page -->
class="..."           <!-- CSS and JS hooks -->
lang="..."            <!-- language of this element's content -->
dir="ltr|rtl|auto"   <!-- text direction; auto for user-generated content -->
hidden                <!-- removes from layout and accessibility tree -->
tabindex="0"          <!-- add to natural tab order -->
tabindex="-1"         <!-- focusable by JS only; not in tab order -->
data-*="..."          <!-- custom data; accessible in JS via el.dataset -->
contenteditable="true|false"
spellcheck="true|false"
translate="yes|no"    <!-- whether translation tools should translate this -->
draggable="true|false"
inert                 <!-- makes element and all descendants non-interactive and invisible to AT -->
autofocus             <!-- focus on load or dialog open; one per page/dialog -->
title="..."           <!-- tooltip on hover; not a replacement for label or alt -->
```

**`inert`** — the correct way to make off-screen content non-interactive without removing it from the DOM. More reliable than manually applying `aria-hidden="true"` and `tabindex="-1"` to every descendant. Use on: navigation drawers, off-screen panels, non-active carousel slides.

**`tabindex` in practice:** only `0` and `-1`. Positive values (`tabindex="1+"`) create a parallel tab order that fights document order and is nearly impossible to maintain.

---

## 12. Metadata and Structured Data

### Open Graph — Full Set

```html
<meta property="og:site_name" content="Site Name">
<meta property="og:type" content="website">
<meta property="og:title" content="Page Title">
<meta property="og:description" content="150–160 character description">
<meta property="og:url" content="https://example.com/page">
<meta property="og:image" content="https://example.com/og-image.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="Description of the OG image">
<meta property="og:locale" content="en_US">
```

OG image: 1200×630px. PNG or JPG — SVG is not reliably supported by social platforms.

### JSON-LD — Structured Data

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Andre",
  "url": "https://archelon.design",
  "jobTitle": "Graphic Designer",
  "sameAs": ["https://linkedin.com/in/..."]
}
</script>
```

JSON-LD is the preferred format for structured data. It sits in `<script>` and doesn't interfere with markup. Schema.org types relevant to portfolio/brand sites: `Person`, `Organization`, `CreativeWork`, `WebSite`, `BreadcrumbList`.

---

## 13. Script Loading — Strategy

```html
<!-- Render-blocking — parse halts until downloaded and executed. Avoid in <head>. -->
<script src="script.js"></script>

<!-- Deferred — parsed in parallel, executed after HTML is parsed, in order -->
<script defer src="main.js"></script>

<!-- Async — parsed in parallel, executed immediately when ready, out of order -->
<script async src="analytics.js"></script>

<!-- Module — deferred by default, strict mode, own scope -->
<script type="module" src="app.js"></script>
```

For a static site: `type="module"` for your own code (deferred automatically). `async` for truly independent third-party scripts with no dependencies. Never render-blocking in `<head>` without `defer` or `async`.

---

## 14. Canonical References

- **MDN HTML Reference** — `developer.mozilla.org/en-US/docs/Web/HTML/Reference` — every element and attribute with examples and browser support
- **WHATWG HTML Living Standard** — `html.spec.whatwg.org` — the actual specification; authoritative on element semantics and parsing
- **WebAIM** — `webaim.org` — practical accessibility, screen reader behaviour, contrast checker
- **ARIA Authoring Practices Guide** — `w3.org/WAI/ARIA/apg` — canonical patterns for ARIA widgets
- **Schema.org** — `schema.org` — structured data vocabulary
- **Open Graph Protocol** — `ogp.me` — OG meta tag specification
