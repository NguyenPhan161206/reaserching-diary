# CSS Frameworks — Bootstrap 5 & Tailwind CSS

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#CSS`, `#Bootstrap`, `#Tailwind`, `#Frameworks`, `#UtilityFirst`, `#Responsive`
**References:** [Bootstrap 5 docs](https://getbootstrap.com/docs/5.3/) · [Tailwind CSS docs](https://tailwindcss.com/docs) · [MDN — CSS first steps](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics) · [MDN — Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** HTML structure ([`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md)); CSS box model ([`02-css-box-model-and-layout-flow.md`](02-css-box-model-and-layout-flow.md)); cascade & specificity ([`03-css-cascade-specificity-inheritance.md`](03-css-cascade-specificity-inheritance.md)); responsive breakpoints ([`06-responsive-web-design.md`](06-responsive-web-design.md))

---

## 📌 Key Takeaways (TL;DR)
- **A CSS framework is pre-written CSS plus a convention.** Bootstrap ships *components* (a ready navbar, card, modal) and a 12-column grid you assemble with class names; Tailwind ships *utilities* (one class = one property) and you compose the component yourself in the markup. Same goal, opposite philosophy.
- **Bootstrap is component-first; Tailwind is utility-first.** Bootstrap answers "give me a card"; Tailwind answers "give me `rounded shadow p-4 bg-white`".
- **Frameworks do not remove CSS — they rename it.** Every Bootstrap `.btn` and every Tailwind `.p-4` is still a declaration block you could have written by hand; the value is consistency, speed, and a maintained reset/layout system.
- **Their responsive breakpoints are media queries with a naming convention.** Bootstrap's `col-md-6` and Tailwind's `md:grid-cols-2` compile to `@media (min-width: …)`. If you understand [`06-responsive-web-design.md`](06-responsive-web-design.md), you already understand the mechanism.
- **For the course you use the CDN link in your Handlebars/EJS layout** — no build step, no SPA. The Joblink project instead compiles Tailwind (utility-first, React/Next.js), so knowing both syntaxes matters.

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

**A CSS framework is pre-written CSS + (optionally) a grid system and JavaScript components.** Writing a production-looking page by hand means re-solving the same problems every time: a reset to kill default margins, a consistent type scale, a way to lay things out in rows and columns, buttons/forms/navbars that look right, and a set of breakpoints. A framework packages those decisions so you inherit someone else's battle-tested defaults.

The two dominant frameworks in this course have **mirror-image philosophies**:

| | **Bootstrap 5** | **Tailwind CSS** |
|---|---|---|
| Unit of composition | **Component** (`.card`, `.navbar`, `.btn`) | **Utility** (`.flex`, `.p-4`, `.text-sm`) |
| You write | A few semantically-named classes | Many single-property classes |
| Ships JavaScript | Yes (modal, dropdown, collapse) | No (pure CSS; you bring your own behavior) |
| Theming | Sass variables / CSS custom properties | `tailwind.config.js` theme |
| Design default | Opinionated "Bootstrap look" | Neutral — you supply the design |

> **Mental model:** think of Bootstrap as buying **IKEA furniture** — flat-pack, pre-cut, you follow the instructions and get a recognizable product. Tailwind is buying **raw timber and a saw** — more work per component, but nothing constrains the final shape. Neither is "better"; they optimize different things (speed to a generic look vs. freedom to a unique one).

**Why this matters:** students usually learn a framework as a bag of magic class names and then hit a wall when a design does not match the framework's opinion. Knowing that both are *just CSS* — with a specificity of `0,1,0` for a single class (see [`03-css-cascade-specificity-inheritance.md`](03-css-cascade-specificity-inheritance.md)) — tells you exactly how to override safely: load your stylesheet *after* theirs, and match their specificity so the cascade resolves in your favor.

---

### 2. Bootstrap 5

#### 2.1 Install — CDN vs npm

**CDN (what the course uses):** drop two `<link>`/`<script>` tags into the layout. Zero tooling, works in a server-rendered page instantly.

```html
<!-- CSS: required -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
      rel="stylesheet" crossorigin="anonymous">
<!-- JS bundle: only needed for interactive components (modal, dropdown, collapse) -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
        crossorigin="anonymous"></script>
```

**npm (production / custom theming):** `npm i bootstrap`, then `import 'bootstrap/dist/css/bootstrap.min.css'` and `import 'bootstrap/dist/js/bootstrap.bundle.min.js'`. The real reason to use npm is **Sass**: you import the source `.scss`, override the variables, and compile your own theme.

```scss
// my-theme.scss — override BEFORE importing Bootstrap
$primary:   #2563eb;
$border-radius: 0.75rem;
@import "node_modules/bootstrap/scss/bootstrap";
```

> **Why it matters:** the CDN is a *fixed, immutable stylesheet*. You can only change its look by re-declaring rules with equal-or-higher specificity afterward. npm + Sass lets you change the *source variables* so every component is themed consistently. The course demo does not need this; a product does.

#### 2.2 The grid — container → row → col

Bootstrap's grid is a **12-column flexbox** wrapped in three layers:

```
.container        → centered, fixed max-width per breakpoint (gutters included)
  .row            → a flex row; cols live ONLY here (negative margins cancel gutters)
    .col-12 .col-md-6 .col-lg-4   → each col spans N/12; wraps to the next line
```

```html
<div class="container">
  <div class="row g-3">                 <!-- g-3 = 1rem gutter on both axes -->
    <div class="col-12 col-md-6 col-lg-4">A</div>
    <div class="col-12 col-md-6 col-lg-4">B</div>
    <div class="col-12 col-lg-4">C</div>
  </div>
</div>
```

Read `col-12 col-md-6 col-lg-4` as a **rule per breakpoint**: full width on phones, half width from `md` (≥768px) up, one-third from `lg` (≥992px) up. It is mobile-first: the unprefixed `col-12` is the default, later classes *add* behavior.

| Breakpoint | Prefix | `min-width` |
|---|---|---|
| extra small | *(none)* | `< 576px` |
| small | `sm` | `576px` |
| medium | `md` | `768px` |
| large | `lg` | `992px` |
| extra large | `xl` | `1200px` |
| extra-extra large | `xxl` | `1400px` |

- **Offsets** push a column right: `offset-md-2` skips two columns at `md` and up.
- **Gutters** are set on the `row` with `g-*` / `gx-*` / `gy-*` (0–5). Do **not** add `mr-*`/`ml-*` to columns — that double-spaces and breaks the negative-margin trick.
- **`container-fluid`** is always full width; `container-md` is full width below `md` and fixed above it.

**How the grid actually works (mechanism).** The `.row` relies on two tricks you can verify in the generated CSS. First, it sets `display: flex; flex-wrap: wrap`, so columns that exceed twelve units in a line wrap to the next line — that is the whole "responsive stacking" behavior. Second, the row applies a **negative horizontal margin** while each column applies an equal **positive horizontal padding**, so the column content lines up flush with the container edge while the padding creates the gutter *inside* each column. This is precisely why you must not add your own `ml-*`/`mr-*` to a column (it breaks the arithmetic) and why a `.col-*` outside a `.row` loses its gutter. The magic number is **12** because it divides evenly by 1, 2, 3, 4, 6, and 12 — every common fraction (halves, thirds, quarters, sixths) has an integer width. You can redefine the breakpoints and the column count with Sass (`$grid-columns`, `$grid-breakpoints`), but for the course the defaults are what you should internalize first.

#### 2.3 Utilities — spacing, text, flex

Utilities are single-purpose classes that override everything at the same specificity.

```html
<div class="d-flex justify-content-between align-items-center p-3 mb-4 bg-light rounded">
  <h2 class="h4 mb-0 text-uppercase text-muted">Dashboard</h2>
  <button class="btn btn-primary btn-sm">New</button>
</div>
```

- **Spacing:** `{property}{side}-{size}` → `m`/`p`, sides `t b s e x y` (start/end, not left/right), size `0–5` plus `auto`. Examples: `mt-3`, `px-4`, `mb-auto`.
- **Text:** `text-center`, `text-muted`, `text-uppercase`, `fw-bold`, `fs-5`.
- **Flex:** `d-flex`, `flex-column`, `justify-content-between`, `align-items-center`, `gap-3`.
- **Display:** `d-none d-md-block` — hidden on phones, block from `md` up. (The classic "show this only on desktop".)

#### 2.4 Key components in one composed example

```html
<!-- navbar -->
<nav class="navbar navbar-expand-lg bg-dark navbar-dark">
  <div class="container">
    <a class="navbar-brand" href="#">Shop</a>
    <button class="navbar-toggler" type="button"
            data-bs-toggle="collapse" data-bs-target="#nav">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="nav">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link active" href="#">Home</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Products</a></li>
      </ul>
    </div>
  </div>
</nav>

<!-- card -->
<div class="card shadow-sm">
  <img src="shoe.png" class="card-img-top" alt="A running shoe">
  <div class="card-body">
    <h5 class="card-title">Running Shoe</h5>
    <p class="card-text text-muted">Lightweight road trainer.</p>
    <button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#buy">Buy</button>
  </div>
</div>

<!-- form control + alert -->
<div class="alert alert-warning" role="alert">Only 3 left in stock!</div>
<label for="qty" class="form-label">Quantity</label>
<input type="number" id="qty" class="form-control" min="1" value="1">

<!-- modal (needs the JS bundle) -->
<div class="modal fade" id="buy" tabindex="-1">
  <div class="modal-dialog"><div class="modal-content">
    <div class="modal-header"><h5 class="modal-title">Confirm</h5></div>
    <div class="modal-body">Place the order?</div>
  </div></div>
</div>
```

The navbar's collapse behavior, the modal open/close, and dropdowns are driven by `data-bs-*` attributes → the JS bundle listens for clicks. **Forget the bundle and the navbar hamburger silently does nothing** — a very common "my Bootstrap is broken" report.

#### 2.5 Theming at a glance

Bootstrap 5 exposes its palette as **CSS custom properties** (`--bs-primary`, `--bs-body-bg`, `--bs-border-radius`), so you can retheme without recompiling:

```css
:root {
  --bs-primary: #2563eb;
  --bs-border-radius: 0.75rem;
}
.btn-primary { --bs-btn-bg: var(--bs-primary); }
```

For a deep theme (different palette everywhere, custom component variants) the Sass route is still the clean way — override `$primary`, `$spacers`, `$grid-breakpoints` and recompile.

---

### 3. Tailwind CSS

#### 3.1 Utilities, not components

Tailwind has **no `.card` or `.btn`**. You build the component by stacking utilities, each mapping to one CSS property:

```html
<article class="max-w-sm rounded-lg shadow-md bg-white p-6">
  <img class="w-full h-40 object-cover rounded" src="shoe.png" alt="A running shoe">
  <h3 class="mt-4 text-lg font-semibold text-gray-900">Running Shoe</h3>
  <p class="mt-1 text-sm text-gray-500">Lightweight road trainer.</p>
  <button class="mt-4 rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
    Buy
  </button>
</article>
```

Common families: `flex`, `grid`, `p-*`/`m-*` (spacing), `text-*` (size/color/align), `bg-*`, `rounded`, `shadow`, `border`, `w-*`/`h-*`, `hover:`/`focus:` variants.

When the predefined scale is not enough, **arbitrary values** in square brackets let you drop to raw CSS without leaving the markup: `w-[37%]`, `bg-[#c026d3]`, `top-[117px]`, `grid-cols-[1fr_2fr]`. A **variant prefix** (`hover:`, `focus:`, `md:`, `dark:`) composes with any utility, and they stack: `md:hover:bg-blue-700` means "at `md` and up, on hover". This composability is the entire grammar of Tailwind — a small set of prefixes multiplied by a large set of utilities.

#### 3.2 Config — extending the design system

`tailwind.config.js` is where the design tokens live. `theme.extend` **adds** to the defaults (keeping them) rather than replacing them:

```js
// tailwind.config.js
export default {
  content: ["./views/**/*.{hbs,ejs,html}", "./src/**/*.js"],
  theme: {
    extend: {
      colors: { brand: { DEFAULT: "#2563eb", dark: "#1e40af" } },
      fontFamily: { sans: ["Inter", "sans-serif"] },
      spacing: { 18: "4.5rem" },
    },
  },
  plugins: [],
};
```

Now `bg-brand`, `text-brand-dark`, and `p-18` exist. **`content` is not optional** — it is the list of files Tailwind scans for class names so it can emit only the classes you use (see the build note in §3.6).

#### 3.3 `@apply` — reusable classes when repetition hurts

When the same twenty utilities repeat, extract them once with `@apply`:

```css
/* input.css */
@layer components {
  .btn-primary {
    @apply rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700;
  }
}
```

Then use `<button class="btn-primary">`. Use it sparingly: over-applying recreates the CSS classes you moved to Tailwind to escape, and it hides the design from the markup. Rule of thumb: extract when the *same* combination appears three or more times.

#### 3.4 Responsive modifiers

Tailwind's responsive prefixes are **mobile-first media queries**. `md:grid-cols-2` means "from the `md` breakpoint *up*", applied as `@media (min-width: 768px)`.

| Prefix | `min-width` | Typical device |
|---|---|---|
| *(none)* | `0` | phone |
| `sm:` | `640px` | large phone |
| `md:` | `768px` | tablet |
| `lg:` | `1024px` | laptop |
| `xl:` | `1280px` | desktop |
| `2xl:` | `1536px` | large desktop |

```html
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">…</div>
```

**This is exactly the mechanism from [`06-responsive-web-design.md`](06-responsive-web-design.md)** — one column by default, two at `sm`, three at `lg` — only written inline instead of in a `@media` block.

#### 3.5 Dark mode

Enable a strategy in the config, then use the `dark:` variant:

```js
// tailwind.config.js
export default { darkMode: "class" };   // or "media" (follows the OS)
```

```html
<body class="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
```

With `"class"`, adding/removing `dark` on `<html>` flips the whole theme — the hook for a manual toggle button.

#### 3.6 Preflight and the CDN-vs-build decision

Tailwind injects **Preflight**, a modern reset (border-box everywhere, margins removed, consistent headings/lists/images). It is the reason a Tailwind page looks "clean" with zero custom CSS.

- **Play CDN** (`<script src="https://cdn.tailwindcss.com"></script>`) compiles utilities *in the browser*. Perfect for a one-file demo, **never for production**: it is large, disables Preflight customization, and can cause a flash of unstyled content.
- **A proper build** (Tailwind CLI or PostCSS) scans `content` files and ships only used classes — often a few KB — and allows `@apply`, config, and plugins.

```bash
npx tailwindcss -i ./input.css -o ./public/styles.css --watch
```

---

### 4. Integration with Server-Rendered Templates

For the course, the server renders HTML (Handlebars/EJS) and the framework is just a stylesheet link in the **shared layout** — no bundler, no SPA. Put the Bootstrap CDN (or the compiled Tailwind output) in the layout's `<head>` so every page inherits it.

```hbs
{{!-- views/layouts/main.hbs (Handlebars) --}}
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link rel="stylesheet" href="/css/app.css">   {{!-- your overrides LAST --}}
  <title>{{title}}</title>
</head>
<body>
  {{> navbar}}                 {{!-- partial full of framework classes --}}
  <main class="container py-4">{{{body}}}</main>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

The same in EJS wrapping other templates:

```ejs
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
<%- include('partials/navbar') %>
```

**Why this works so well with SSR:** the classes are plain strings in the template; there is no build step, the browser downloads one stylesheet, and the HTML still renders without JavaScript. See [`../../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md) for the layout/partial mechanism.

---

### 5. Choosing — Bootstrap vs Tailwind vs Custom CSS

| Dimension | Bootstrap 5 | Tailwind CSS | Custom CSS |
|---|---|---|---|
| Philosophy | Component-first, opinionated | Utility-first, unopinionated | Exactly what you write |
| Learning curve | Low — copy a snippet, it works | Moderate — must know CSS *and* hundreds of class names | High for the first design, then grows with the project |
| Design consistency | Automatic (one look everywhere) | Yours to keep consistent via config tokens | Yours to keep — or lose |
| Overhead | ~200KB CSS + ~80KB JS (CDN) | Only used classes (build), or huge via Play CDN | As small as you make it |
| Change a design | Fight the framework / recompile | Edit classes (or config) | Edit your rules |
| When it wins | Admin tools, prototypes, "get a professional UI fast", teams that don't want design decisions | Custom product UIs, design systems, React/Next teams, Joblink | Tiny pages, deep learning, or when the framework's weight isn't justified |

**Decision rule for a 4th-year student:** use **Bootstrap** when speed-to-competent-UI matters and the default look is acceptable (course project). Use **Tailwind** when you want a bespoke look without leaving the markup, and you are ready to keep a config as the single source of design truth (Joblink). Reach for **custom CSS** for a blog page or when practicing the fundamentals — everything a framework does, you can still do by hand.

---

## 🧾 Command / Code Cheat Sheet

| Construct | What it does | When to use | Example |
|---|---|---|---|
| `container` / `container-fluid` | centered max-width wrapper / always full width | the outermost grid wrapper | `<div class="container">…</div>` |
| `row` + `g-3` | flex row + gutter spacing | any horizontal group of columns | `<div class="row g-3">` |
| `col-12 col-md-6 col-lg-4` | 12/6/4 of 12 columns per breakpoint | responsive card grids | stacks → half → third |
| `offset-md-2` | skip N columns at a breakpoint | center or indent a column | `<div class="col-md-4 offset-md-4">` |
| `d-none d-md-block` | hide below `md`, show from `md` up | desktop-only blocks | `<aside class="d-none d-lg-block">` |
| `m-* / p-* / px-* / py-*` | margin / padding, 0–5 | tweaking spacing without custom CSS | `mb-3`, `px-4` |
| `d-flex justify-content-between align-items-center` | flex row, spread, vertically centered | navbars, toolbars | header layouts |
| `navbar navbar-expand-lg` | responsive navbar with collapse | site navigation | paired with `navbar-toggler` |
| `card` / `card-body` / `card-title` | a bordered content panel | product/stat containers | `card shadow-sm` |
| `data-bs-toggle` / `data-bs-target` | wire a button to a JS component | open a modal/dropdown/collapse | `data-bs-toggle="modal"` |
| `flex grid grid-cols-3 gap-4` (TW) | flexbox / CSS grid helpers | modern layout in markup | `grid grid-cols-1 lg:grid-cols-3` |
| `p-4 m-2 text-sm bg-white rounded shadow` (TW) | one-property utilities | composing a component | a card built by hand |
| `sm: md: lg:` (TW) | mobile-first breakpoint prefix | responsive utilities | `md:flex-row` |
| `dark:` (TW) | apply in dark mode | dark themes | `dark:bg-gray-900` |
| `@apply` (TW) | inline utilities into a CSS class | extract a repeated combo | `.btn { @apply px-4 py-2; }` |

---

## 🧪 Practical Lab — Build the Same UI Twice

> Goal: internalize the **philosophy difference** by producing one responsive navbar + one card grid in each framework. Save as `bootstrap.html` and `tailwind.html`; open both in a browser and resize.

**Lab 1 — Bootstrap version**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</head>
<body>
  <nav class="navbar navbar-expand-lg bg-dark navbar-dark">
    <div class="container">
      <a class="navbar-brand" href="#">Shop</a>
      <button class="navbar-toggler" data-bs-toggle="collapse" data-bs-target="#nav">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="nav">
        <ul class="navbar-nav ms-auto">
          <li class="nav-item"><a class="nav-link" href="#">Home</a></li>
          <li class="nav-item"><a class="nav-link" href="#">Products</a></li>
        </ul>
      </div>
    </div>
  </nav>
  <main class="container py-4">
    <div class="row g-4">
      <div class="col-12 col-md-6 col-lg-4"><div class="card p-3">Card A</div></div>
      <div class="col-12 col-md-6 col-lg-4"><div class="card p-3">Card B</div></div>
      <div class="col-12 col-lg-4"><div class="card p-3">Card C</div></div>
    </div>
  </main>
</body>
</html>
```

**Lab 2 — Tailwind version (Play CDN)**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50">
  <nav class="bg-gray-900 text-gray-100">
    <div class="mx-auto flex max-w-5xl items-center justify-between p-4">
      <a class="text-lg font-bold" href="#">Shop</a>
      <ul class="flex gap-6">
        <li><a class="hover:text-blue-400" href="#">Home</a></li>
        <li><a class="hover:text-blue-400" href="#">Products</a></li>
      </ul>
    </div>
  </nav>
  <main class="mx-auto max-w-5xl p-4">
    <div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
      <div class="rounded-lg bg-white p-4 shadow">Card A</div>
      <div class="rounded-lg bg-white p-4 shadow">Card B</div>
      <div class="rounded-lg bg-white p-4 shadow">Card C</div>
    </div>
  </main>
</body>
</html>
```

**Expected output — desktop (≥992px):** dark navbar with brand left and two links right; a **3-column** card row (Tailwind: `lg:grid-cols-3`).
**Expected output — tablet (768–991px):** Bootstrap shows a hamburger icon (links collapse); cards are **2 columns** then wrap.
**Expected output — mobile (<576px):** navbar collapses to the hamburger in Bootstrap; both frameworks show **1 column** of cards, full width.

*What to notice:* identical pixels, opposite code. Bootstrap hides the *behavior* (collapse) behind a component + JS; Tailwind makes you write the layout but hands you zero opinion and +0 JS.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---|---|---|
| Mixing framework + custom CSS without understanding specificity | your rule "doesn't apply" or gets overwritten | load your stylesheet **after** the framework; match specificity (`0,1,0`) and rely on cascade order ([`03-`](03-css-cascade-specificity-inheritance.md)) |
| Reaching for `!important` against the framework | brittle CSS you can never override later | increase specificity or fix source order instead |
| Missing Bootstrap JS bundle | navbar hamburger, modal, dropdown do nothing | include `bootstrap.bundle.min.js` before `</body>` |
| Columns not direct children of `.row` | gutters/negative margins look wrong | `.col-*` must be inside `.row`, `.row` inside a `.container` |
| Adding `ml-*`/`mr-*` to columns | double spacing, overflow | use the row's `g-*` gutter classes |
| CDN utility classes not purged / Play CDN in production | multi-MB CSS, flash of unstyled content | use a Tailwind build with `content`; use the Bootstrap CSS CDN (not Play CDN) |
| Forgetting `<meta name="viewport">` | "mobile" layout shows a shrunken desktop | add the viewport tag ([`06-`](06-responsive-web-design.md)) |
| Tailwind classes missing in output | styles vanish in production | the class must appear literally in a `content` file; dynamic strings like `` `bg-${c}` `` are not detected |
| Using `col-md-6` alone and expecting mobile width | column stays auto/full unexpectedly | always include the unprefixed base (`col-12 col-md-6`) |
| Overusing `@apply` | you rebuild utility classes as named classes | extract only after 3+ repeats |

---

## 🧪 Practice Tasks

> No solutions. Build, then ask the AI to review your markup and screenshots.

- [ ] **Task 1 — Bootstrap product grid.** Build a responsive 4-card product grid (1 col mobile → 2 tablet → 4 desktop) using `container`/`row`/`col-*`, add an `alert` above it, and a `modal` opened by a "Details" button. Verify the modal opens (JS bundle present).
- [ ] **Task 2 — Tailwind rebuild.** Recreate Task 1 in Tailwind (Play CDN is fine). Then extract the repeated card markup into an `@apply` component class and explain in one sentence why you would or would not keep the extraction.
- [ ] **Task 3 — Theming.** In Bootstrap, retheme `.btn-primary` to a custom color using CSS custom properties (no Sass). In Tailwind, add a `brand` color via `theme.extend` and use `bg-brand`. Which took fewer keystrokes, and which scales to ten components?
- [ ] **Task 4 — Specificity fight.** With Bootstrap loaded, add a custom `.btn { background: purple; }` and explain why nothing changed. Then fix it by source order, then by specificity. Write down which mechanism the cascade used (see [`03-`](03-css-cascade-specificity-inheritance.md)).

## 🔗 Tech Cross-References

- **CSS cascade & specificity** — every framework class is specificity `0,1,0`; overriding them is a cascade problem, not a framework problem. [`03-css-cascade-specificity-inheritance.md`](03-css-cascade-specificity-inheritance.md)
- **Responsive Web Design** — Bootstrap's breakpoints and Tailwind's `md:` prefixes *are* mobile-first media queries; this note is the packaged form of that one. [`06-responsive-web-design.md`](06-responsive-web-design.md)
- **Template engines (Handlebars/EJS)** — the CDN link and framework classes live in the shared layout and partials; SSR means no build step. [`../../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md)
- **Joblink (Tailwind + Next.js)** — the real code base that uses utility-first CSS, and why it pairs naturally with component-based rendering. 🔗 [github.com/qminh77/joblink](https://github.com/qminh77/joblink)

## 📝 Research Journey

- **Why:** The course requires **both** a component framework and a utility framework, and the Joblink project I want to read uses Tailwind. I already understand the cascade and media queries, so I came in wanting the *philosophy* behind the class names, not just a copy-paste catalogue.
- **Struggle:** At first Bootstrap and Tailwind looked like the same thing spelled differently. The crack in that idea was trying to change one button color: in Bootstrap I was fighting someone else's decision, in Tailwind I *was* the decision. That is when "component-first vs utility-first" stopped being jargon.
- **Aha moment:** Realizing `col-md-6` and `md:grid-cols-2` both compile to `@media (min-width: 768px)`. Frameworks are not a new layout system — they are a naming convention over the media queries and specificity rules I already learned in notes 03 and 06.
- **Career link:** As an AI Engineer building economics/business dashboards, most of my output is a UI consuming a model's JSON. A design system (Bootstrap for internal tools, Tailwind for customer-facing products) is what makes a prototype look trustworthy enough to put in front of a stakeholder.

## 🔗 Related Topics

- [`03-css-cascade-specificity-inheritance.md`](03-css-cascade-specificity-inheritance.md) — the mechanism every framework override depends on.
- [`06-responsive-web-design.md`](06-responsive-web-design.md) — breakpoints, mobile-first, and the viewport tag these frameworks package.
- [`08-forms-fetch-and-multipart-upload.md`](08-forms-fetch-and-multipart-upload.md) — framework form controls meet the browser upload flow, next.
- [`../../web_server/05-template-engines-handlebars-ejs-layout.md`](../web_server/05-template-engines-handlebars-ejs-layout.md) — where the CDN link actually goes in an SSR app.
- [SUGGESTED] `bridge-design-tokens.md` — bridge note: how `theme.extend` tokens in Tailwind, CSS custom properties in Bootstrap, and a Figma style guide are the *same* design-token idea expressed three ways.
- [SUGGESTED] `09-accessibility-and-aria.md` — frameworks ship accessible-ish components, but keyboard focus, labels, and ARIA still need deliberate work; worth its own note soon.

## 🤔 Open Questions

- [ ] When does the utility-class markup become harder to maintain than a component class, and is there a measurable threshold (class-count per element)?
- [ ] How does Tailwind's JIT `content` scanning behave with server-generated class names in Handlebars partials — do dynamic templates risk purging styles?
- [ ] Is Bootstrap's shipped JavaScript a liability in an SSR app that already has server logic, or does it usefully save hand-written client code?
- [ ] For a business dashboard, do we gain more from Bootstrap's opinionated consistency or from Tailwind's design freedom + a shared config?

---
*Frameworks are conventions, not new CSS. Learn the mechanism, and you can leave any framework without losing the skill.*
