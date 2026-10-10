# HTML Structure & Semantics — The Structure Layer of the Web

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#HTML`, `#Semantics`, `#DOM`, `#Accessibility`, `#WebBasics`
**References:** [MDN — HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element), [MDN — Global attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes), [MDN — HTML forms guide](https://developer.mozilla.org/en-US/docs/Learn/Forms), [web.dev — Learn HTML](https://web.dev/learn/html), [WHATWG HTML Living Standard](https://html.spec.whatwg.org/multipage/)
**Status:** Draft
**Difficulty:** Beginner

---

## 📌 Key Takeaways (TL;DR)

- HTML is the **structure layer**: it declares *what each thing is*. CSS decides how it looks; JavaScript decides how it behaves. A tag is syntax, an **element** is the tag plus content plus attributes, an **attribute** is configuration.
- **Semantic elements** (`header`, `nav`, `main`, `article`, …) exist so machines — search engines, screen readers, future frameworks — can understand the page instead of reverse-engineering it from class names.
- The browser is **error-tolerant**: broken markup still renders. That is why HTML bugs hide silently, and why the live DOM in DevTools — not "View Source" — is the source of truth.
- `alt` on images and `name` on form fields are not decoration: `alt` is the accessible alternative, `name` is the key the server actually receives.
- HTML is parsed into a **DOM tree**. That single tree is what CSS styles and what JavaScript mutates — nothing else on the web works without it.

---

## 🧠 Detailed Notes

### 1. Core Concepts

**1.1 The three layers.** A web page is built from three cooperating technologies:

| Layer | Technology | Question it answers |
|-------|-----------|---------------------|
| Structure | **HTML** | *What is this thing?* (a heading, a list, an image) |
| Presentation | **CSS** | *How should it look?* (colour, size, position) |
| Behaviour | **JavaScript** | *What should it do when I interact?* (click, type, fetch) |

An analogy that survives scrutiny: HTML is the skeleton, CSS is the skin and clothing, JS is the muscles and nervous system. The point of separating them is *change isolation* — you can restyle a whole page without touching a single word of content, because the content never encoded its own appearance.

**1.2 Tags vs elements vs attributes.** These three words are used loosely in conversation but they are distinct:

- **Tag** — the markup token itself, e.g. `<a>` (opening) or `</a>` (closing).
- **Element** — the complete unit: opening tag + content + closing tag.
- **Attribute** — a `key="value"` pair written inside the opening tag.

```html
<a href="https://example.com" target="_blank" rel="noopener">Example</a>
<!--  ^open tag  ^attr 1            ^attr 2        ^attr 3    ^content ^close tag -->
```

The whole line is an `<a>` **element**; `href`, `target`, `rel` are **attributes**. This matters because when someone says "add an attribute," they mean the opening tag — and when you inspect an element, `attribute` values are *static text* while *properties* on the DOM node are live (a distinction that becomes important the moment JavaScript reads them, see [`04-javascript-language-core.md`](./04-javascript-language-core.md)).

**1.3 Void (empty) elements.** Some elements can never have content or children, so they have no closing tag: `<br>`, `<img>`, `<input>`, `<hr>`, `<meta>`, `<link>`, `<source>`. Writing `<img src="a.png" />` (the trailing slash) is legal but *optional* — it comes from XHTML and means nothing in HTML. Writing `<img>...</img>` is simply invalid.

**1.4 Global attributes.** Attributes valid on *every* element:

| Attribute | Purpose | Gotcha |
|-----------|---------|--------|
| `id` | Unique identifier for one element | Must be unique per document; used by `#fragment` links and `label for` |
| `class` | Space-separated style/behaviour hooks | Reusable; the main CSS selector surface |
| `title` | Advisory tooltip text | Not a substitute for visible labels; touch devices ignore it |
| `data-*` | Custom data, e.g. `data-user-id="42"` | Read by JS via `dataset.userId`; keeps DOM free of non-standard attributes |
| `hidden` | Boolean, removes from rendering | Not the same as CSS `display:none` semantics for a11y in every engine |
| `lang` | Language of the content | Screen readers switch voice/pronunciation based on it |
| `style` | Inline CSS | Legal but discouraged — breaks the layer separation |

**1.5 Boolean attributes.** Some attributes are true by *presence*, false by *absence* — they have no meaningful value:

```html
<input type="checkbox" checked />        <!-- checked -->
<input type="checkbox" checked="false" /> <!-- STILL checked! presence wins -->
<input type="text" required disabled />   <!-- required + disabled -->
```

This trips up beginners constantly. `disabled="false"` does **not** enable an element; to disable the behaviour you must remove the attribute entirely.

**1.6 The document skeleton.** Every page shares the same skeleton:

```html
<!DOCTYPE html>                                  <!-- standards mode, not quirks -->
<html lang="en">                                 <!-- lang = a11y + translation -->
<head>
  <meta charset="UTF-8" />                       <!-- character encoding FIRST -->
  <meta name="viewport"
        content="width=device-width, initial-scale=1.0" /> <!-- mobile scaling -->
  <title>Page title — shown in the browser tab</title>
  <link rel="stylesheet" href="styles.css" />    <!-- presentation -->
  <script src="app.js" defer></script>           <!-- defer: run after parse -->
</head>
<body>
  <!-- everything the visitor sees lives here -->
</body>
</html>
```

Why each line matters: `<!DOCTYPE html>` switches the browser out of *quirks mode* (a legacy rendering mode that changes box sizing — see [`02-css-box-model-and-layout-flow.md`](./02-css-box-model-and-layout-flow.md)). `charset="UTF-8"` must appear within the first 1024 bytes or the browser guesses the encoding and mangles non-ASCII text. `lang` drives screen-reader pronunciation. The viewport meta tag is the single reason mobile pages are not rendered as tiny desktop pages. `defer` lets the script run after the HTML is parsed without blocking rendering.

### 2. Semantic Structure

**2.1 Landmark elements.** HTML5 added elements whose *names describe their role*. Because they map to accessibility landmarks, both assistive technology and search engines get meaning for free.

| Element | Role | When to use |
|---------|------|-------------|
| `<header>` | Introductory content / branding | Page masthead, or the top of an `<article>` |
| `<nav>` | Major navigation block | Sets of links (site menu, breadcrumbs, ToC) |
| `<main>` | The dominant, unique content | Exactly **once** per page |
| `<section>` | Thematic grouping, needs a heading | A chapter of a page |
| `<article>` | Self-contained composition | Blog post, comment, product card — stand-alone in a feed |
| `<aside>` | Tangentially related content | Sidebars, pull quotes, "related posts" |
| `<footer>` | Closing content | Copyright, contact, site links |
| `<figure>` + `<figcaption>` | Media with a caption | Image, chart, code listing with a caption |

The test for `<article>` vs `<section>`: could you syndicate it into an RSS feed on its own and have it still make sense? If yes → `<article>`. If it only groups related content → `<section>`.

**2.2 Heading hierarchy.** Headings are the page's outline, and they are the *primary navigation affordance* for screen reader users (who jump heading-to-heading like you jump by `Ctrl+F`).

```html
<h1>Web Application Development</h1>   <!-- one per page: the page's subject -->
  <h2>Client-side basics</h2>
    <h3>HTML</h3>
    <h3>CSS</h3>
  <h2>Server-side basics</h2>
    <h3>Express</h3>
```

Rules: do not skip levels (`h1` straight to `h3`), and never choose a heading level for its *size* — size is CSS's job. A heading is a structural claim, and using it for visual styling corrupts the outline.

**2.3 Why semantics matter.** Four concrete payoffs:

1. **SEO** — crawlers weight `<h1>`, `<nav>`, `<article>` when deciding what a page is about.
2. **Accessibility** — screen readers can list landmarks ("navigation region, main region") only if you use the real elements.
3. **Maintainability** — `<nav>` communicates intent to the next human. `<div class="nav">` is a guess.
4. **Future frameworks** — React, Next.js, and Vue ultimately render to these same tags. A component that emits semantic markup is testable and styles predictably; a component that emits div-soup forces every consumer to re-learn its private class conventions. The Joblink project's JSX is readable precisely because it stayed semantic.

**2.4 DIV-soup vs semantic markup.** The anti-pattern, then the fix:

```html
<!-- ❌ DIV-SOUP: no meaning, only class names carry intent -->
<div class="header">
  <div class="nav"><div class="nav-item"><a href="/">Home</a></div></div>
</div>
<div class="content">
  <div class="post">
    <div class="title">Hello</div>
    <div class="text">Body text…</div>
  </div>
</div>
```

```html
<!-- ✅ SEMANTIC: the outline is built into the tags -->
<header>
  <nav><ul><li><a href="/">Home</a></li></ul></nav>
</header>
<main>
  <article>
    <h1>Hello</h1>
    <p>Body text…</p>
  </article>
</main>
```

Both render almost identically. Only the second one is *understood* — by a crawler, a screen reader, a teammate, and by any tool that later wants to extract the article.

**2.5 Text elements.** `<p>` paragraphs; `<a href>` anchors (`target="_blank"` opens a new tab but always pair it with `rel="noopener"` to stop the opened page from controlling yours); `<strong>` = *importance* and `<em>` = *stress emphasis*, as opposed to the purely presentational `<b>` (bold) and `<i>` (italic) — prefer `strong`/`em` because they survive a CSS redesign. `<span>` is the generic inline container with no meaning, the inline twin of `<div>`. `<br>` forces a line break (use sparingly; never as a spacing hack). `<blockquote cite>` for quotations, `<code>`/`<pre>` for code, `<hr>` for a thematic break.

**2.6 Images.** `<img>` is a void element and its `alt` attribute is **not optional**:

```html
<img src="chart-revenue-2026.png"
     alt="Bar chart: revenue grew from $1.2M in Q1 to $3.4M in Q4 2026"
     width="640" height="360"
     loading="lazy" />
<img src="divider.svg" alt="" role="presentation" />  <!-- decorative → empty alt -->
```

`alt` is read aloud by screen readers and shown when the image fails to load; a decorative image gets `alt=""` so it is *skipped* rather than announced. Always set `width`/`height` (or an aspect ratio) so the browser reserves space and the text does not jump when the image arrives (this is the layout-shift metric `CLS`).

**2.7 Lists.** `<ul>` unordered, `<ol>` ordered, `<dl>` description lists. Lists are among the most frequently abused elements: navigation menus, feature grids, and step sequences are *semantically lists* even when CSS hides the bullets.

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
</ul>

<ol start="3">                <!-- numbering continues at 3 -->
  <li>Install Node</li>
  <li>Create the server</li>
</ol>

<dl>
  <dt>HTML</dt><dd>The structure layer.</dd>
  <dt>CSS</dt><dd>The presentation layer.</dd>
</dl>
```

**2.8 Tables — for real tabular data only.** A table is a 2D relationship between row headers, column headers, and data cells. `<th scope="col">` / `<th scope="row">` tell assistive tech which header governs a cell. `<caption>` gives the table an accessible name.

```html
<table>
  <caption>Quarterly revenue (USD millions)</caption>
  <thead>
    <tr><th scope="col">Quarter</th><th scope="col">Revenue</th></tr>
  </thead>
  <tbody>
    <tr><th scope="row">Q1</th><td>1.2</td></tr>
    <tr><th scope="row">Q2</th><td>2.6</td></tr>
  </tbody>
</table>
```

Never use `<table>` to lay out a page (the classic 2000s sin). Tables force content into a grid that is meaningless to `header/nav/main`, break responsiveness, and lie to a screen reader that announces "table with 3 rows" for what is really a sidebar.

### 3. Forms (intro)

Forms are the input boundary between the browser and the server. This is an introduction; the full flow lives forward in the server track.

```html
<form action="/subscribe" method="post">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required autocomplete="email" />

  <label for="plan">Plan</label>
  <select id="plan" name="plan">
    <option value="free">Free</option>
    <option value="pro" selected>Pro</option>
  </select>

  <label for="message">Message</label>
  <textarea id="message" name="message" rows="4"></textarea>

  <button type="submit">Subscribe</button>
</form>
```

Two attributes carry almost all the meaning:

- **`id` + `label for`** — clicking the label focuses the field, and a screen reader announces the label with the input. This is the accessibility contract.
- **`name`** — this is the key the server *actually receives*. The `id` connects DOM nodes to each other; the `name` connects the form to the request body. A field with no `name` is invisible to the server even if it looks perfect on screen. (When this form posts, the browser sends `email=...&plan=pro&message=...` — the server track picks it up in the forms note.)

**GET vs POST:**

| | `method="get"` | `method="post"` |
|--|----------------|-----------------|
| Data location | Appended to the URL (`?email=…`) | Sent in the request body |
| Visible / bookmarkable | Yes | No |
| Length limit | Yes (URL length) | Effectively none |
| Idempotent | Should be (safe to repeat) | No |
| Use for | Search, filters, pagination | Creating/mutating data, login, uploads |

The rule of thumb: use `GET` when the request *reads*, `POST` when it *changes* state or carries secrets.

### 4. How It Works (Mechanism)

The browser's HTML parser turns a byte stream into a tree in a fixed pipeline:

```
raw bytes
  → characters      (decoded using <meta charset>)
  → tokens          ("<p>", "text", "</p>")
  → nodes           (element / text nodes)
  → DOM tree        (elements nested in document order)
```

The parser is **error-tolerant by design**. Unlike a JSON parser that rejects malformed input, the HTML parser has deterministic recovery rules baked into the spec:

- An unclosed `<li>` is auto-closed when the next `<li>` or the list ends.
- A `<p>` is implicitly closed when a block element starts inside it.
- Missing `<html>`, `<head>`, or `<body>` are *inserted automatically*.
- Text directly inside a `<table>` is "foster-parented" out of the table to keep the grid valid.

That tolerance is a gift and a trap: your page *looks* fine while the real tree differs from what you wrote.

A full runnable semantic page:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Reading List</title>
</head>
<body>
  <header>
    <h1>My Reading List</h1>
    <nav aria-label="Primary">
      <ul><li><a href="#books">Books</a></li></ul>
    </nav>
  </header>

  <main>
    <section id="books" aria-labelledby="books-heading">
      <h2 id="books-heading">Books</h2>
      <article>
        <h3>Thinking, Fast and Slow</h3>
        <p>By Daniel Kahneman. <em>Essential</em> for understanding bias.</p>
        <img src="kahneman.jpg" alt="Cover of Thinking, Fast and Slow" width="120" height="180" />
      </article>
    </section>

    <aside aria-label="About this list">
      <p>Curated for an AI-in-business curriculum.</p>
    </aside>
  </main>

  <footer><p>© 2026</p></footer>
</body>
</html>
```

Its DOM tree (simplified) is a set of nested nodes:

```
html
├── head
│   ├── meta[charset]
│   ├── meta[viewport]
│   └── title
└── body
    ├── header
    │   ├── h1  "My Reading List"
    │   └── nav → ul → li → a "Books"
    ├── main
    │   ├── section#books
    │   │   ├── h2 "Books"
    │   │   └── article
    │   │       ├── h3 "Thinking, Fast and Slow"
    │   │       ├── p
    │   │       └── img
    │   └── aside
    └── footer → p "© 2026"
```

Why it matters: this tree is the interface everything else uses. CSS matches selectors against it; JavaScript queries and mutates it (`document.querySelector`) — see [`05-dom-and-browser-event-loop.md`](./05-dom-and-browser-event-loop.md). Crucially, **"View Source" shows the original text; DevTools → Elements shows the parsed tree.** When the two disagree, the tree is what actually rendered, because the parser "fixed" your markup.

### 5. Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Invalid nesting (block inside inline, e.g. `<span><div>…</div></span>`) | Layout splits unexpectedly; parser silently reorganises | Keep block elements outside inline ones; check DevTools tree |
| Unclosed tags | Auto-closed in the wrong place; siblings nest wrongly | Close every non-void tag; validate with the W3C validator |
| Skipped heading levels (`h1` → `h3`) | Broken a11y outline; confusing navigation | Only descend one level at a time |
| Missing `alt` | Screen reader announces the filename or nothing | Always provide `alt`, empty for decorative images |
| Useless divs on everything | No landmarks, unreadable JSX | Use semantic elements first, `<div>` last |
| Table used for layout | Rigid, non-responsive; false a11y announcements | Use Flexbox/Grid (see the CSS note) |
| Duplicate `id` | `#fragment` and `label for` bind to the wrong node | Keep `id` unique per document |
| `<br>` used for spacing | Cannot be restyled; breaks on reflow | Use CSS `margin`/`gap` |

### Comparison Table — Semantic tag vs div-and-class equivalent

| Semantic tag | Div equivalent | What the semantic version buys |
|--------------|----------------|--------------------------------|
| `<header>` | `<div class="header">` | Landmark role for AT; no class needed |
| `<nav>` | `<div class="nav">` | Announced as a navigation region |
| `<main>` | `<div class="content">` | "Skip to main content" targets it directly |
| `<article>` | `<div class="post">` | Self-contained unit for feeds/search |
| `<aside>` | `<div class="sidebar">` | Complementary landmark |
| `<footer>` | `<div class="footer">` | Contentinfo landmark |
| `<figure>` + `<figcaption>` | `<div class="img-wrap">` | Caption programmatically tied to the media |
| `<button>` | `<div class="btn">` | Keyboard focus, Enter/Space activation, role |

The pattern is consistent: a div carries *styling only*; a semantic element carries *styling plus meaning* the browser can act on.

---

## 🔗 Tech Cross-References

- **JavaScript & the DOM** — HTML is the *data*; the DOM is the live object tree JavaScript reads and mutates. Every framework exists to manage that mutation. [→ `04-javascript-language-core.md`](./04-javascript-language-core.md)
- **CSS** — CSS selects against the exact DOM tree the parser produced, which is why "fixing" HTML changes which styles match. [→ `02-css-box-model-and-layout-flow.md`](./02-css-box-model-and-layout-flow.md)
- **Databases** — every `<table>`/list you render is ultimately rows from a query; the `name` attributes on a form become the parameters a server hands to the database layer. [→ `../../databases/README.md`](../databases/README.md)
- **YAML / config** — `package.json` and CI config are markup-ish data files with the same "structure vs values" separation HTML has between elements and attributes; the parsing-trap instinct transfers. [→ `../../dev_foundation/yaml_config/README.md`](../yaml_config/README.md)

## 📝 Research Journey

- **Why:** I could already read React/JSX in the Joblink codebase, but I realised I was pattern-matching class names rather than understanding what the markup *meant*. I wanted to stop guessing.
- **Struggle:** The error tolerance genuinely confused me — I assumed invalid HTML would error like Java or Python. The idea that the parser silently *rewrites* my page, and that "View Source" could disagree with the DOM, was the hardest mental shift.
- **Aha moment:** When I first used the DevTools accessibility tree and saw my div-soup page announce nothing useful, while the semantic version exposed "navigation / main / article" landmarks. Semantics stopped being a style rule and became a machine-readable API.
- **Career link:** As an AI Engineer building business tools, my front-ends will be consumed by crawlers, screen readers, and automated tests. Clean semantic DOM is the substrate that pipelines and models extract from; a machine-readable page is half of good data.

## 🔗 Related Topics

- [`03-css-cascade-specificity-inheritance.md`](./03-css-cascade-specificity-inheritance.md) — once markup exists, *why a style doesn't apply* is the next confusion.
- [`06-responsive-web-design.md`](./06-responsive-web-design.md) — semantic content reflowed across screen sizes.
- [`07-css-frameworks-bootstrap-tailwindcss.md`](./07-css-frameworks-bootstrap-tailwindcss.md) — frameworks wrap the same tags; utility classes are styling, not structure.
- [SUGGESTED] **`bridge-rendering-pipeline.md`** — trace bytes → DOM → CSSOM → layout → paint end-to-end. *Why:* it unifies the HTML, CSS, and DOM notes into one mental movie. *Folder:* `web_basics/`.
- [SUGGESTED] **`accessibility-and-aria-basics.md`** — ARIA roles, labels, and keyboard navigation. *Why:* semantics is the foundation of a11y, but real products need the ARIA layer on top. *Folder:* `web_basics/`.

## 🤔 Open Questions

- [ ] When is a `<div>` actually the *right* answer over inventing a class-named semantic element? Where is the line?
- [ ] How exactly does foster parenting decide where stray table text lands — can I predict the resulting tree?
- [ ] Does using `<section>` everywhere (even without headings) hurt, or is it neutral for SEO?
- [ ] How do React's custom components map back to these tags at hydration — is the DOM always semantic in practice?

---
*Structure first. If the skeleton is honest, every layer above it can be rebuilt without breaking the meaning.*
