# Web Basics — HTML, CSS, JavaScript, DOM

**Purpose:** Read a code base and understand *how a browser turns text into pixels and reacts to you*. Mechanism-first, not "become a front-end developer".

**Template:** **Template A (Tech Note, adapted)** for mechanism notes (01–03, 05), **Template G** for JavaScript-as-a-language usage (04).

## Scope — what "sơ" means here

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
| — | *(none yet)* | — | — |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | HTML structure & semantics | The tag vocabulary you need to read any project |
| High | CSS box model & layout flow | How the browser distributes space (block/inline/flex/grid) |
| High | Cascade, specificity & inheritance | Why your style "doesn't apply" — the #1 CSS confusion |
| Medium | JavaScript language core (types, scope, functions, arrays) | Reading any modern code base |
| Medium | DOM & browser event loop | How clicks become code, and why animations jank |
| Low | Forms & accessibility semantics | Needed to read real projects, not to write them |
| Low | Fetch/API calls (ties into PostgREST) | Connects web → database |

---

## 🔗 Tech Cross-References (Section K)
- **DOM ↔ JavaScript** — HTML is the *data*; the DOM is the *live object tree* JS mutates. Every framework exists to manage that mutation.
- **CSS ↔ data layout** — flexbox/grid are layout algorithms, closer to `display: contents` thinking than to art; same instinct as arranging a data table
- **PostgREST** — the server returns JSON; this is the layer that turns it into elements
- **Databases** — every list you see is rows rendered by a template; pagination is a database problem before it is a UI problem
- **Tailwind / Joblink** — utility classes are a *different syntax for the same CSS concepts*; reading Tailwind means understanding specificity first
- **YAML** — `package.json` is config you did not choose; same parsing traps as YAML

## 🔗 Related Topics
- [`../databases/04-postgrest-auto-rest-api.md`](../databases/04-postgrest-auto-rest-api.md) — the API this layer consumes
- [`../databases/05-schema-design-and-data-modeling.md`](../databases/05-schema-design-and-data-modeling.md) — where the data comes from
- 🔗 [Joblink](https://github.com/qminh77/joblink) — the real code base to read these notes against (Next.js/React/Tailwind/Supabase)
- [SUGGESTED] `bridge-rendering-pipeline.md` — bridge note: HTML → DOM → CSSOM → layout → paint, one end-to-end trace

---
*Created: 2026-09-28 — Planned templates: A (mechanism) + G (JS language usage)*
