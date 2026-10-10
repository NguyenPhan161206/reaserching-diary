# Web Basics — HTML, CSS, JavaScript, DOM

**Purpose:** Read a code base and understand *how a browser turns text into pixels and reacts to you*. Mechanism-first, not "become a front-end developer".

**Template:** **Template A (Tech Note, adapted)** for mechanism notes (01–03, 05, 06), **Template G (Dev Tool & Command Note)** for hands-on language/framework notes (04, 07, 08).

> **Track:** this is the **web track**. JavaScript is the language *here* — it is deliberately **not** a DSA language (that track is Java + C++, see [`../dsa/`](../dsa/README.md)). The JS notes below are written from the browser's point of view; for algorithms, JavaScript's missing heap and shallow stack are documented in [`../dsa/01-learning-roadmap.md`](../dsa/01-learning-roadmap.md).

## Scope — what "just enough" means here

> Learn the tags, the drawing/effect/layout-distribution mechanism, and enough JavaScript to follow the logic. Not pixel-perfect CSS, not a framework.

| Layer | Goal | Explicitly NOT the goal |
|-------|------|------------------------|
| **HTML** | Understand tags, attributes, the DOM tree, semantics, forms | Mastering every HTML5 API |
| **CSS** | Box model, layout flow, flex/grid distribution, cascade & specificity, animation triggers | Design systems, Tailwind mastery |
| **JavaScript** | Types, scope, functions, arrays/objects, modules, `async`/`await`, event listeners | Framework internals, build tooling |
| **Browser** | Rendering pipeline, reflow vs repaint, event loop, DevTools reading | Browser engine implementation |

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | HTML structure & semantics | A | What tags exist, what each one *means*, and why the DOM tree matters? |
| 02 | CSS box model & layout flow | A | How does the browser distribute space (block / inline / flex / grid)? |
| 03 | Cascade, specificity & inheritance | A | Why does my style "not apply", and who wins when two rules conflict? |
| 04 | JavaScript language core | G | What is the actual type/scope/async model I am writing against? |
| 05 | DOM & browser event loop | A | How does a click become code, and why do animations jank? |
| 06 | Responsive web design | A | How do breakpoints, units and media queries adapt one layout to every screen? |
| 07 | CSS frameworks: Bootstrap & Tailwind | G | How does a utility-class framework give me the cascade for free — and cost me what? |
| 08 | Forms, Fetch & multipart upload | G | How does browser-to-server data actually travel (form POST, JSON, FormData)? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| Medium | Web accessibility & ARIA beyond the basics | Note 01 touches semantics; real projects (Joblink) need the full checklist |
| Medium | CSS animations & transitions triggers | Note 05 explains *why* animations jank — the tricks that avoid it come next |
| Low | Client-side state: modules, bundlers & npm | The step between vanilla JS and reading a Next.js code base |
| Low | Browser rendering pipeline deep dive (CRP) | The current notes cover layout-flow; the full paint pipeline is the next layer |

---

## 🔗 Tech Cross-References (Section K)
- **DOM ↔ JavaScript** — HTML is the *data*; the DOM is the *live object tree* JS mutates. Every framework exists to manage that mutation.
- **JavaScript ↔ Java/C++** — same algorithm, different container model (`Map`/`Set` are insertion-ordered and hide-classed; Java's `HashMap` is unordered; C++'s `vector` is contiguous). Useful contrast when porting a DSA idea into web code → [`../dsa/README.md`](../dsa/README.md)
- **CSS ↔ data layout** — flexbox/grid are layout algorithms, closer to `display: contents` thinking than to art; same instinct as arranging a data table
- **PostgREST** — the server returns JSON; this is the layer that turns it into elements
- **Databases** — every list you see is rows rendered by a template; pagination is a database problem before it is a UI problem
- **Tailwind / Joblink** — utility classes are a *different syntax for the same CSS concepts*; reading Tailwind means understanding specificity first
- **YAML** — `package.json` is config you did not choose; same parsing traps as YAML

## 🔗 Related Topics
- [`../databases/04-postgrest-auto-rest-api.md`](../databases/04-postgrest-auto-rest-api.md) — the API this layer consumes
- [`../databases/05-schema-design-and-data-modeling.md`](../databases/05-schema-design-and-data-modeling.md) — where the data comes from
- [`../web_server/README.md`](../web_server/README.md) — the server half of the request/response loop (SSR templates consume these pages)
- [`../dsa/README.md`](../dsa/README.md) — the other track (Java + C++), and why JS is not in it
- 🔗 [Joblink](https://github.com/qminh77/joblink) — the real code base to read these notes against (Next.js/React/Tailwind/Supabase)
- [SUGGESTED] `bridge-rendering-pipeline.md` — bridge note: HTML → DOM → CSSOM → layout → paint, one end-to-end trace

---
*Created: 2026-09-28 — Updated: 2026-10-10 — Templates: A (mechanism) + G (JS/language usage)*
