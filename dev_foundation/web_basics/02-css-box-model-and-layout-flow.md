# CSS Box Model & Layout Flow — How the Browser Distributes Space

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#CSS`, `#BoxModel`, `#Layout`, `#Flexbox`, `#WebBasics`
**References:** [MDN — CSS Box Model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model), [MDN — Flexbox guide](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox), [MDN — CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout), [web.dev — Learn CSS Layout](https://web.dev/learn/css/layout), [CSS Tricks — A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
**Status:** Draft
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)

- Every element is a **box** with four nested layers: content → padding → border → margin. The box model is the arithmetic that turns your `width: 300px` into the pixels actually drawn.
- `box-sizing: content-box` (the default) makes `width` mean *content only*, so padding and border are **added on top** — the single most common reason width-based layouts overflow. Set `box-sizing: border-box` globally.
- **Normal flow** is the browser's default: block boxes stack vertically, inline boxes flow along a line. **Margin collapse** (vertical margins merging) is the surprise that follows.
- **Flexbox** distributes space along *one* axis (row or column) — it answers "align and grow these items in a line." **Grid** controls *two* axes at once — it answers "place these items on a page."
- Positioning (`relative`, `absolute`, `fixed`, `sticky`) takes a box *out of* or *partly out of* normal flow; `z-index` only compares elements in the same stacking context.

---

## 🧠 Detailed Notes

### 1. Core Concepts — The Box Model

Every rendered element is a rectangular box with four layers, from inside out:

```
┌──────────────────────────── margin ────────────────────────────┐
│  ┌───────────────────────── border ─────────────────────────┐  │
│  │  ┌──────────────────── padding ──────────────────────┐   │  │
│  │  │                                                   │   │  │
│  │  │                  CONTENT                          │   │  │
│  │  │            (text, image, child box)               │   │  │
│  │  │                                                   │   │  │
│  │  └───────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

- **Content** — the actual text/image/child boxes.
- **Padding** — inner space between content and border; takes the element's background colour.
- **Border** — the line around padding.
- **Margin** — outer space *between this box and its neighbours*; always transparent.

**The width gotcha.** By default, `width` refers to the **content box only**:

```css
.card {
  width: 300px;
  padding: 20px;
  border: 5px solid #333;
  /* default box-sizing: content-box */
}
```

The element does not occupy 300px — it occupies:

$$\text{outer width} = m_l + b_l + p_l + w + p_r + b_r + m_r = 0 + 5 + 20 + 300 + 20 + 5 + 0 = 350\text{px}$$

So two such cards side by side try to be 700px inside a 680px container — and overflow. The fix is `border-box`, where the declared `width` is the *border box*:

```css
*, *::before, *::after { box-sizing: border-box; }  /* global reset — do this on day one */

.card { width: 300px; padding: 20px; border: 5px solid #333; }
/* now the box is exactly 300px wide; content becomes 300 - 40 - 10 = 250px */
```

With `border-box`, the content width is derived:

$$w_{\text{content}} = w_{\text{declared}} - p_l - p_r - b_l - b_r$$

Why it matters: `border-box` makes width arithmetic *additive and predictable*, which is why every professional codebase applies it globally. Without it, "50% + 50%" never fits because padding pushes the total past 100%.

### 2. Normal Flow & Display

**Normal flow** is what happens when you specify no positioning: block boxes stack top-to-bottom, inline boxes flow left-to-right and wrap.

| `display` | Line behaviour | Width/height | Vertical margin | Typical use |
|-----------|----------------|--------------|-----------------|-------------|
| `block` | Starts on a new line, fills available width | Respected | Respected (and collapses) | `div`, `p`, `section`, layout containers |
| `inline` | Flows within a line of text | **Ignored** (sizes to content) | Ignored vertically | `span`, `a`, `strong` |
| `inline-block` | Flows like inline, but sizes like a block | Respected | Respected | Buttons, chips, badges |
| `none` | Removed from rendering entirely | — | — | Toggle visibility |

The key trap: setting `width`/`height`/vertical `margin` on an `inline` element does nothing. That is why navigation links that need a hit area are set to `inline-block` or wrapped in a flex container.

**Margin collapse.** Two vertical margins that meet merge into the *larger* of the two, not their sum. Three cases matter:

1. **Adjacent siblings** — a `<p>` with `margin-bottom: 20px` followed by one with `margin-top: 30px` sit 30px apart, not 50px.
2. **Parent and first/last child** — a child's top margin can "escape" through its parent and move the parent instead, *if* there is nothing (border/padding/inline content) separating them.
3. **Empty blocks** — a block with no content/height/padding/border has its own top and bottom margins collapse together.

```css
/* Demonstration of parent-child collapse */
.outer { background: lightblue; }
.inner { margin-top: 40px; }          /* this pushes OUTER down, not INNER within it */
```

The fixes all work by giving the parent a *boundary* or a new formatting context, so the child's margin cannot pass through:

```css
.outer { display: flow-root; }        /* modern, explicit: establishes a BFC */
/* alternatives: */
/* .outer { overflow: hidden; }       creates a BFC (side effects: clips overflow) */
/* .outer { padding-top: 1px; }       any padding stops the collapse */
/* .outer { display: flex; }          flex/grid children never collapse margins */
```

Why it matters: margin collapse is invisible in your CSS but visible in the layout, so it produces "why did everything shift down?" bugs that no amount of staring at the element explains. `display: flow-root` is the intention-revealing fix.

### 3. Positioning

The `position` property decides whether a box follows normal flow or is lifted out of it.

| Value | In flow? | Offsets (`top/right/bottom/left`) measured from | Common use |
|-------|----------|--------------------------------------------------|-----------|
| `static` | Yes (default) | Ignored | Normal content |
| `relative` | Yes (keeps its space) | Its **own** normal position | Small nudges; the positioning anchor for children |
| `absolute` | **No** (removed) | Nearest **positioned ancestor** (else the page) | Dropdowns, tooltips, badges |
| `fixed` | **No** | The **viewport** (does not scroll) | Sticky headers, modal overlays |
| `sticky` | Yes, until threshold | Toggles between relative and fixed | Sticky section headings |

```css
/* relative is often used only to make a parent an anchor for absolute children */
.card { position: relative; }                 /* anchor */
.badge {
  position: absolute; top: 8px; right: 8px;   /* pinned to .card's corner */
  background: crimson; color: white;
}
```

`absolute` looks for the nearest ancestor with `position` other than `static`; if none exists it falls back to the initial containing block (the page). This is *the* classic dropdown bug — a menu anchors to the whole page because a parent forgot `position: relative`.

**`z-index` at a glance:** it only orders elements that are in the **same stacking context**. Something like `position: absolute; z-index: 9999;` cannot rise above an element inside a parent that has `opacity < 1`, `transform`, or `filter` — those create a *new* stacking context, and children are trapped inside it. The practical rule: reach for the smallest `z-index` that works, and establish explicit stacking contexts on purpose.

### 4. Flexbox (deep)

Flexbox is a **one-dimensional** layout engine: you pick an axis, and it distributes space and alignment along that axis.

**Axes.** With the default `flex-direction: row`, the **main axis** runs horizontally and the **cross axis** vertically. Switch to `column` and they swap. Every flex property is named for the axis, not for the screen direction — this is the single most important idea in flexbox.

**Container properties:**

```css
.container {
  display: flex;              /* establishes the flex formatting context */

  flex-direction: row;        /* main axis: row | row-reverse | column | column-reverse */

  justify-content: space-between;  /* distribute along MAIN axis:
                                      flex-start | flex-end | center |
                                      space-between | space-around | space-evenly */

  align-items: center;        /* align items along CROSS axis:
                                 stretch (default) | flex-start | flex-end | center | baseline */

  flex-wrap: wrap;            /* allow items to wrap onto new lines: nowrap (default) | wrap */

  align-content: flex-start;  /* spacing between WRAPPED LINES (only matters with wrap):
                                 flex-start | flex-end | center | space-between | space-around | stretch */

  gap: 12px;                  /* modern spacing: row-gap + column-gap, no margin hacks */
}
```

**Item properties:**

```css
.item {
  flex-grow: 0;    /* how eagerly to consume FREE space (default 0 = don't grow) */
  flex-shrink: 1;  /* how eagerly to give up space when overflow (default 1) */
  flex-basis: auto;/* the starting size before growing/shrinking: auto | 0 | <length> | <percentage> */

  align-self: flex-end; /* override the container's align-items for THIS item */
  order: 2;             /* visual order (default 0); use sparingly — it does NOT change tab order */
}
```

**The `flex` shorthand** encodes grow, shrink, and basis in one declaration:

| Shorthand | Expands to | Meaning |
|-----------|-----------|---------|
| `flex: 1` | `1 1 0%` | Equal shares; ignore content size |
| `flex: auto` | `1 1 auto` | Grow/shrink, but start from content size |
| `flex: none` | `0 0 auto` | Fixed at content size; never grow or shrink |
| `flex: 0 1 200px` | `0 1 200px` | Start at 200px, shrink under pressure, never grow |

The difference between `flex: 1` (`basis: 0%`) and `flex: auto` (`basis: auto`) is the classic confusion: with `flex: 1`, all items start from zero and split space *equally*; with `flex: auto`, a long-content item starts bigger, so equal `grow` does not mean equal width.

**Classic centering patterns:**

```css
/* Pattern A: center on BOTH axes */
.center-both {
  display: flex;
  justify-content: center;  /* main axis (horizontal in a row) */
  align-items: center;      /* cross axis (vertical in a row) */
  height: 100vh;            /* flex needs a height to center within */
}

/* Pattern B: center a single child with auto margins — no justify/align needed */
.center-auto { display: flex; }
.center-auto > .child { margin: auto; }  /* auto absorbs all free space on every side */

/* Pattern C: push the last item to the far end of the row */
.toolbar { display: flex; align-items: center; }
.toolbar .spacer { margin-left: auto; }
```

Runnable demonstration with expected appearance:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flexbox labs</title>
  <style>
    .row {
      display: flex;
      align-items: center;
      gap: 8px;
      background: #eef;
      padding: 8px;
      margin-bottom: 12px;
    }
    .box { padding: 12px; background: #6cf; color: white; }

    /* 1. space-between along the main axis */
    .between { justify-content: space-between; }
    /* 2. one item grows to fill the leftover space */
    .grow > .box.growable { flex: 1; text-align: center; }
    /* 3. wrap onto new lines when the row runs out of width */
    .wrapping { flex-wrap: wrap; max-width: 220px; }
  </style>
</head>
<body>
  <!-- Expected: A pinned left, B pinned right -->
  <div class="row between"><div class="box">A</div><div class="box">B</div></div>

  <!-- Expected: the middle box stretches to absorb all free space -->
  <div class="row grow">
    <div class="box">Left</div><div class="box growable">GROW</div><div class="box">Right</div>
  </div>

  <!-- Expected: four boxes; the 4th wraps to a second line -->
  <div class="row wrapping">
    <div class="box">1</div><div class="box">2</div>
    <div class="box">3</div><div class="box">4</div>
  </div>
</body>
</html>
```

### 5. CSS Grid (intro)

Grid is **two-dimensional**: it lays out items in rows *and* columns simultaneously, and the parent defines the tracks.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);  /* 3 equal columns (1fr = 1 fraction of free space) */
  grid-template-rows: auto 1fr auto;      /* header / stretchable body / footer */
  gap: 16px;
}
```

Items are **auto-placed** into the next free cell in reading order. For deliberate layouts, name the areas:

```css
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  gap: 16px;
}
.page > header  { grid-area: header; }
.page > aside   { grid-area: sidebar; }
.page > main    { grid-area: main; }
.page > footer  { grid-area: footer; }
```

The `1fr` unit is the key primitive: it distributes *remaining* space after fixed tracks and gaps, so `1fr 1fr 1fr` with `gap` always fits the container with no `content-box` arithmetic.

**Decision table — Flexbox vs Grid:**

| Dimension | Flexbox | Grid |
|-----------|---------|------|
| Axes | One (row **or** column) | Two (rows **and** columns) |
| Sizing logic | Content-driven ("where do the items want to go?") | Layout-driven ("place items into this map") |
| Track definition | Implicit, emergent | Explicit (`grid-template-*`) |
| Best for | Toolbars, nav bars, card rows, centering, equal-height columns | Whole-page layouts, galleries, dashboards, any 2D map |
| Wrapping | Items wrap to new lines but no column alignment across lines | Rows and columns stay aligned across the whole grid |
| Reorder | `order`, `flex-direction` | `grid-area`, explicit placement |
| Overlap | Not natural | Easy (place items in the same cells + `z-index`) |

Rule of thumb: **"one direction → flex, two directions → grid."** They compose — a grid cell whose contents are a flex row is the normal arrangement.

### 6. Path Forward — Frameworks Wrap This

Bootstrap's grid is a thin class layer over *exactly* these ideas: `.row` sets a flex container, `.col-*` sets `flex-basis`/`flex-grow` percentages, and `gap`/breakpoints mirror `flex-wrap`. Tailwind exposes the primitives directly (`flex`, `justify-between`, `items-center`, `grid-cols-3`, `gap-4`). Neither replaces the box model or the axes — they just save typing. If you understand `flex-basis` and `1fr`, Tailwind reads like plain CSS; if you do not, every utility class is a magic spell. See [`07-css-frameworks-bootstrap-tailwindcss.md`](./07-css-frameworks-bootstrap-tailwindcss.md).

---

## 🔗 Tech Cross-References

- **JavaScript & the DOM** — the layout engine measures the *live* DOM; changing a class from JS can force a reflow, which is where animation jank comes from. [→ `04-javascript-language-core.md`](./04-javascript-language-core.md)
- **DOM & the event loop** — reflow (layout) and repaint are synchronous work caused by DOM mutations; this is why "measure then write, don't interleave" is a performance rule. [→ `05-dom-and-browser-event-loop.md`](./05-dom-and-browser-event-loop.md)
- **Databases** — a data table's columns and a CSS `grid-template-columns` are both *schemas for arranging cells*; the mental model of aligning rows transfers directly. [→ `../../databases/README.md`](../databases/README.md)
- **DSA** — flex distribution is a space-partitioning algorithm (grow weights, shrink priorities) not unlike a weighted allocation problem; fr units are a ratio split. [→ `../dsa/README.md`](../dsa/README.md)

## 📝 Research Journey

- **Why:** I kept hitting layouts that "looked right in my head" but broke the moment I set a width, and I realised I was memorising CSS instead of understanding the arithmetic underneath it.
- **Struggle:** Margin collapse was the hardest — an invisible rule that moves a parent because of a child's margin, with no CSS line mentioning the parent at all. It made me distrust my own mental model until I learned to think in formatting contexts.
- **Aha moment:** `box-sizing: border-box` plus `display: flow-root` removed two whole classes of bug in one afternoon. Realising the box model is *literal arithmetic* — not a vibe — made the rest of layout predictable.
- **Career link:** AI-in-business products are dashboards: charts, tables, toolbars, data cards. Layout is not decoration here — a dashboard that reflows correctly at any width is how users trust the numbers. Flexbox and grid are the tools that make model output legible.

## 🔗 Related Topics

- [`03-css-cascade-specificity-inheritance.md`](./03-css-cascade-specificity-inheritance.md) — the *other* half of "why doesn't my style apply": the box model explains space, the cascade explains precedence.
- [`06-responsive-web-design.md`](./06-responsive-web-design.md) — media queries switching flex directions and grid templates across breakpoints.
- [`07-css-frameworks-bootstrap-tailwindcss.md`](./07-css-frameworks-bootstrap-tailwindcss.md) — how Bootstrap `.col` and Tailwind utilities wrap `flex-basis`/`1fr`.
- [SUGGESTED] **`bridge-rendering-pipeline.md`** — where layout sits between style resolution and paint, and why "layout thrashing" costs frames. *Why:* connects this arithmetic to the browser's performance model. *Folder:* `web_basics/`.
- [SUGGESTED] **`css-animation-and-transforms-basics.md`** — `transform`, `transition`, and which properties are cheap to animate. *Why:* the course project needs hover/transition polish, and `transform` also creates stacking contexts. *Folder:* `web_basics/`.

## 🤔 Open Questions

- [ ] When does `display: flow-root` behave differently from `overflow: hidden` for stopping margin collapse, beyond the overflow-clipping side effect?
- [ ] Is `flex: 1` on many items ever *worse* than `flex-basis` percentages for predictable cross-browser sizing?
- [ ] What exactly triggers a new stacking context — is there a definitive list I can memorise for `z-index` debugging?
- [ ] At what complexity does CSS Grid stop being worth it over a flex row plus `flex-wrap` for a card gallery?

---
*Layout is arithmetic you can see. Master the box, the axes, and the formatting context, and the pixels stop surprising you.*
