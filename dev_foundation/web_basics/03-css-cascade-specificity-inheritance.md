# CSS Cascade, Specificity & Inheritance — "Why Doesn't My CSS Apply?"

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#CSS`, `#Cascade`, `#Specificity`, `#Inheritance`
**References:** [MDN — Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Cascade) · [MDN — Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity) · [MDN — Inheritance](https://developer.mozilla.org/en-US/docs/Web/CSS/Inheritance) · [web.dev — The cascade](https://web.dev/learn/css/the-cascade)
**Status:** Draft
**Difficulty:** Intermediate

---

## 📌 Key Takeaways (TL;DR)
- When two rules set the **same property on the same element**, the browser resolves the conflict with a fixed algorithm: **origin & importance → inline styles → cascade layers → specificity → source order**. "The last line in my file wins" is only the *final tie-breaker*, not the whole story.
- Specificity is a 4-tuple $(a,b,c,d)$ = (inline, IDs, classes/attributes/pseudo-classes, elements/pseudo-elements). It is compared column-by-column, **not added up**. One ID beats any number of classes.
- Only *some* properties **inherit** (`color`, `font-*`, `line-height`); layout/box properties (`margin`, `padding`, `border`, `width`, `height`, `background`) do **not**. A child's value is either inherited or its own initial value — never "whatever the parent looked like".
- The four CSS-wide keywords `inherit` / `initial` / `unset` / `revert` give you explicit control over that default, property by property.
- `@layer` orders entire *groups* of rules by author intent instead of by selector count — the sanctioned modern alternative to the `!important` arms race.

## 🧠 Detailed Notes

### 1. Core Concepts — the cascade is the conflict resolver

CSS is *declarative*: you describe what should be true, and the browser figures out how. The trouble is that many declarations can target the same element, so the browser needs a **deterministic referee**. That referee is the **cascade**.

Think of it as a sort: every declaration that applies to a given element enters a list, the browser sorts that list by a fixed priority key, and the **highest-sorted declaration wins** — per property, independently. `color` and `margin` are decided separately; a rule can win for one property and lose for another.

Three **origins** produce declarations, and they are *not* equal:

| Origin | Who writes it | Example |
|--------|---------------|---------|
| **User-agent (UA)** | the browser vendor | default `body { margin: 8px }`, `h1 { font-size: 2em }`, `ul { padding-left: 40px }` |
| **User** | the reader (rare, e.g. accessibility styles, reader mode) | a custom stylesheet forcing large fonts |
| **Author** | *you*, the page developer | your `<style>` blocks and linked `.css` files |

For **normal** declarations the precedence rises UA → user → author (your styles beat the browser's). For **`!important`** declarations the whole order **reverses** (a user `!important` beats your `!important`), which is exactly why `!important` is dangerous: it removes your say over the page and hands the top slot to the user.

**Why this matters:** the single most common bug report — "I set `color: red` and it's still blue" — is almost always *not* a typo. It is the cascade picking a higher-priority declaration (a framework rule, an inline style, or a stale `!important`) that you did not notice.

### 2. Specificity — scoring a selector

Once origin and importance are tied, the browser compares **specificity**: a count of how "loud" each selector is. Write it as a four-part score $(a,b,c,d)$:

- $a$ — inline `style="..."` declarations (usually written separately as "higher than any selector").
- $b$ — the number of **ID** selectors (`#lead`).
- $c$ — the number of **classes**, **attribute** selectors (`[type="text"]`), and **pseudo-classes** (`:hover`, `:nth-child()`).
- $d$ — the number of **elements** and **pseudo-elements** (`p`, `div`, `::before`).

Comparison is **lexicographic**, left to right — the way you compare version numbers, *not* the way you add integers:

$$(b_1 > b_2) \;\Rightarrow\; (b_1,c_1,d_1) > (b_2,c_2,d_2) \quad\text{regardless of } c,d$$

So $(\ldots,1,0,0)$ always beats $(\ldots,0,\infty,\infty)$: **one ID outranks any number of classes**. The universal selector `*` and inherited values both score $(0,0,0,0)$ — they never *outrank* an explicit rule; they only act as defaults.

| Selector | Specificity $(a,b,c,d)$ | Notes |
|----------|------------------------|-------|
| `*` | $(0,0,0,0)$ | universal — zero, always loses ties |
| `li` | $(0,0,0,1)$ | one element |
| `ul li` | $(0,0,0,2)$ | two elements |
| `li::before` | $(0,0,0,2)$ | element + pseudo-element |
| `.note` | $(0,0,1,0)$ | one class |
| `li.note` | $(0,0,1,1)$ | class + element |
| `a:hover` | $(0,0,1,1)$ | `:hover` is a pseudo-**class** → column $c$ |
| `input[type="text"]` | $(0,0,1,1)$ | attribute selector → column $c$ |
| `#lead` | $(0,1,0,0)$ | one ID |
| `#lead .note p` | $(0,1,1,1)$ | ID + class + element |
| `div#lead.note.active` | $(0,1,2,1)$ | ID + two classes + element |
| `style="color:red"` | $(1,0,0,0)$ | inline — beats any selector |
| `:where(.note)` | $(0,0,0,0)$ | **`:where()` contributes zero** by definition |
| `:is(#lead, .note)` | $(0,1,0,0)$ | `:is()` uses the **most specific** argument |

**Worked example.** Given `<p id="lead" class="note">`, these rules all target `color`:

```
p { color: green; }              /* (0,0,0,1) */
.note { color: blue; }           /* (0,0,1,0)  beats p          */
#lead { color: purple; }         /* (0,1,0,0)  beats .note      */
```

Winner: `purple`. Replacing the ID with another class would not help — $c$ is compared only after $b$ ties at 0, and here $b$ does not tie. This is why "just add class after class" never beats an ID.

**Gotcha:** never treat specificity as a base-10 number (`.a.b` = 20, `#id` = 100). That mental model breaks the moment column $c$ reaches double digits — `.a.b.c.d.e.f.g.h.i.j.k` (11 classes) must **not** be imagined as beating an ID. The tuple comparison is the truth.

### 3. The Cascade Step-by-Step — how a conflict resolves

The real sorting key, from **strongest to weakest**:

1. **Origin + importance.** The combined buckets, weakest first:
   `UA normal` → `user normal` → `author normal` → `animations` → `author !important` → `user !important` → `UA !important` → `transitions`.
   So a normal author rule beats all normal UA/user rules, but **any** `!important` author rule beats a normal inline style.
2. **Inline (element-attached) styles.** Among declarations of the same origin/importance bucket, a `style="..."` declaration beats every selector — even an ID.
3. **Cascade layers** (`@layer`). Later layers beat earlier layers for normal rules; the order **reverses for `!important`** rules.
4. **Specificity.** The $(a,b,c,d)$ score from §2.
5. **Source order.** If everything above ties, **the declaration that appears later wins** (across all stylesheets, in link order).

**Same property declared twice, resolving both ways:**

```css
/* Case A — different specificity: specificity decides, order is irrelevant */
.note { color: blue; }   /* (0,0,1,0) */
p     { color: green; }  /* (0,0,0,1) ... but the class rule always wins */
/* Result: blue, even though the element rule came later. */

/* Case B — equal specificity: source order decides */
.note  { color: teal; }
.note  { color: navy; }
/* Result: navy — same (0,0,1,0), the later declaration wins. */
```

**Why it matters:** this ordering explains two things that feel arbitrary otherwise. First, why a normal `#id { }` rule still loses to `p { color: red !important }`. Second, why reordering *files* can silently change your page: a framework loaded *after* your CSS wins every specificity tie by source order.

### 4. Inheritance — which properties flow down

Inheritance is a **separate mechanism** from the cascade. It decides the *starting value* for properties you did not set: if a property is inheritable and the element has no winning declaration for it, the element uses its **parent's computed value**.

| Typically **inherit** | Typically **do NOT inherit** |
|-----------------------|------------------------------|
| `color` | `margin`, `padding` |
| `font-family`, `font-size`, `font-style`, `font-weight` | `border`, `outline` |
| `line-height` | `width`, `height` |
| `letter-spacing`, `word-spacing` | `background` |
| `text-align`, `visibility`, `cursor` | `display`, `position`, `overflow` |
| `list-style-*` | `top`/`left`/`right`/`bottom` |

The intuition: properties that describe **text style and readability** inherit (you want a paragraph to keep the body font), while properties that describe **the box** do not (you do not want every `<p>` inside a card to inherit the card's `border`).

**Explicit control — the four CSS-wide keywords:**

| Keyword | Meaning | Use it when |
|---------|---------|-------------|
| `inherit` | force the property to take the parent's computed value | a normally non-inherited prop should flow down: `button { border: inherit; }` |
| `initial` | use the property's **spec-defined initial value** (not inheritance) | reset regardless of context: `p { color: initial; }` → black |
| `unset` | inherit if the property is inheritable, otherwise initial | generic "drop my declaration" without checking the property's rules |
| `revert` | roll back to the value from the **previous origin** (usually UA/user) | undo your author styles to restore browser defaults: `a { all: revert; }` |

`all` is a shorthand for every property at once, so `all: unset` is the sledgehammer that strips an element's inherited styles entirely.

**The inherited-value-as-default gotcha.** A developer writes `h2 { color: red; }`, sees all child `<span>`s turn red, and concludes "`color` is always the parent's". Not quite — the spans inherit `color` because it is inheritable *and* nothing overrides it. Now they set `span { margin: 20px; }` expecting the same trick and nothing happens: `margin` does not inherit. Inheritance is a **per-property** contract, not a universal rule. A related trap: form controls (`<button>`, `<input>`) get UA declarations for `font` and `color`, so they *do not* inherit your page font by default — a classically ugly button.

### 5. `@layer` (modern) — ordering by intent, not by score

The problem `@layer` solves: without it, the only levers are specificity and `!important`. To keep a theme override winning, people escalate selectors (`#app .card .title`) until the file becomes unmaintainable, then reach for `!important`, and the cascade collapses.

```css
@layer base, components, utilities;

@layer base {
  a { color: navy; }                  /* feel free to use low specificity here */
}
@layer components {
  .btn { color: white; }              /* beats base, even mid-selector */
}
@layer utilities {
  .text-red { color: red; }           /* beats components, by layer order */
}
```

Rules in a **later** layer win over earlier layers **regardless of specificity**, so you can keep every selector simple. For `!important` declarations the layer order **reverses** — earlier layers' important rules win — which gives framework authors a predictable escape hatch.

**Why it matters:** this is how modern frameworks organize themselves. Tailwind v4 ships its reset, components, and utilities as layers so a user's own styles can sit in a later layer and win without `!important`. The pattern is: **earn priority by ordering, not by fighting with selector weight.**

### 6. Runnable demo — element vs class vs `!important`

Save as `cascade-demo.html`, open in a browser, and read the rendered color against the comments.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Cascade demo</title>
  <style>
    /* specificity (0,0,0,1) */
    p { color: green; }

    /* specificity (0,0,1,0) — beats the element rule above */
    .note { color: blue; }

    /* specificity (0,1,0,0) — beats the class rule */
    #lead { color: purple; }

    /* author !important — this bucket outranks ALL normal declarations,
       so it also beats the ID rule and the normal inline style below */
    p { color: red !important; }
  </style>
</head>
<body>
  <!-- inline style is (1,0,0,0) but a NORMAL inline style still
       loses to the p{...!important} rule above -->
  <p id="lead" class="note" style="color: orange;">
    I render RED: !important beats inline-normal, ID, class and element.
  </p>

  <!-- Remove the !important line above and this one becomes ORANGE:
       inline (1,0,0,0) beats the #lead ID rule (0,1,0,0). -->
  <p class="note" style="color: orange;">
    With the !important removed, I render ORANGE (inline wins).
  </p>

  <!-- Inheritance demo: color flows down, border does not. -->
  <div style="color: teal; border: 2px solid teal; padding: 8px;">
    Direct text is teal, and the border is on the div only.
    <p>I inherit color (teal) but <strong>not</strong> border or padding.</p>
    <button>I do NOT inherit the teal color — the UA sets button text.</button>
  </div>
</body>
</html>
```

Expected computed results: the first `<p>` is **red**; remove the `!important` rule and it turns **orange** (inline beats ID). The nested `<p>` is teal; the `<button>` is black-ish, because the UA stylesheet declares its color.

### 7. Comparison Table — selectors vs specificity

| # | Selector | $(a,b,c,d)$ | Beats |
|---|----------|-------------|-------|
| 1 | `*` | $(0,0,0,0)$ | nothing (defaults only) |
| 2 | `ul li` | $(0,0,0,2)$ | `*`, single elements |
| 3 | `.note` | $(0,0,1,0)$ | any element-only selector |
| 4 | `a:hover` | $(0,0,1,1)$ | `.note`, elements |
| 5 | `input[type="text"]` | $(0,0,1,1)$ | `.note`, elements |
| 6 | `div.note.active` | $(0,0,2,1)$ | single-class selectors |
| 7 | `:where(#lead)` | $(0,0,0,0)$ | nothing — argument ignored |
| 8 | `#lead` | $(0,1,0,0)$ | any number of classes |
| 9 | `div#lead .note p` | $(0,1,1,2)$ | `#lead` alone |
| 10 | `style="..."` | $(1,0,0,0)$ | every selector (normal rules) |

### 8. Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Overspecifying (`#a .b .c .d p`) | a one-word change requires `!important` | use flat class selectors; keep specificity low by policy |
| Specificity arms race | ever-longer selectors, then `!important` everywhere | adopt `@layer` or a naming convention (BEM); stop competing |
| `!important` abuse | nothing can override the rule; themes break | reserve it for accessibility/utility overrides; prefer layers |
| Styling all descendants (`.card p`) | nested components get unexpected styles | use the child combinator `.card > p`, or scope by class |
| Forgetting the UA reset | mystery `8px` gap around the page, odd list indentation | add a reset/`normalize`; Tailwind's Preflight does this |
| Expecting margins to inherit | child spacing does not match parent's | set spacing explicitly; `margin`/`padding` never inherit |
| Inline style wins silently | "my class does nothing" on an element with `style=""` | inspect the element; JS often writes inline styles |
| `:where()` seems ignored | a wrapping selector has no effect | `:where()` is intentionally 0-specificity — one class beats it |

### 9. Why It Matters — the bridge to frameworks

Every CSS framework is a **pre-made cascade**. Bootstrap ships several thousand declarations, and its grid/utility classes are engineered with specific specificity so they layer predictably; that is why a Bootstrap class can override your custom rule when loaded later or written with equal specificity. Tailwind goes further: its **Preflight** is a documented UA-style reset (the `body { margin: 8px }` and heading sizes you keep fighting), and Tailwind's utility classes deliberately sit at specificity $(0,0,1,0)$ — one class — precisely so they do not create arms races; conflicts are resolved with layer order and the occasional `!` modifier. Read [`../07-css-frameworks-bootstrap-tailwindcss.md`](./07-css-frameworks-bootstrap-tailwindcss.md) with this note in hand, and "why does the framework's style win?" stops being magic.

## 🔗 Tech Cross-References
- **[CSS box model & layout flow]** — the cascade decides *which* declaration wins; the box model decides what that winning declaration *does* to space. The UA stylesheet is also the source of the default `body`/heading margins you fight here. [→ `../02-css-box-model-and-layout-flow.md`](./02-css-box-model-and-layout-flow.md)
- **[CSS frameworks (Bootstrap, Tailwind)]** — their classes are ordinary declarations competing in the same cascade; Preflight is a reset, and Tailwind v4 uses `@layer`. [→ `../07-css-frameworks-bootstrap-tailwindcss.md`](./07-css-frameworks-bootstrap-tailwindcss.md)
- **[JavaScript language core]** — `element.style.color = "red"` writes an **inline** style $(1,0,0,0)$, while `element.classList.toggle("active")` only changes a class $(0,0,1,0)$; this asymmetry explains why JS inline styling so often overrides your stylesheet. [→ `../04-javascript-language-core.md`](./04-javascript-language-core.md)
- **[Databases]** — both the cascade and a query planner are *declarative resolvers*: you state intent, and an engine picks a winner under precedence rules you must understand to predict the outcome. [→ `../../databases/README.md`](../databases/README.md)

## 📝 Research Journey
- **Why:** I kept hitting "why doesn't this style apply?" and reaching for `!important` as a reflex. I wanted to understand the rule instead of randomly escalating.
- **Struggle:** The ordering was the hard part — I had internalized "last rule wins" and could not explain why a top-of-file `#id` rule was still losing.
- **Aha moment:** Specificity is a **tuple compared left to right**, not a sum. Once I stopped adding "100 + 10", the ID-vs-class behaviour became obvious.
- **Career link:** Reading a real React/Tailwind codebase (Joblink) means tracing why a class loses to another class. The cascade is the mental model behind every "style not applying" ticket, and it is a prerequisite for writing ML dashboards that render correctly.

## 🔗 Related Topics
- [../01-html-structure-and-semantics.md](./01-html-structure-and-semantics.md) — the elements and semantics these selectors target
- [../02-css-box-model-and-layout-flow.md](./02-css-box-model-and-layout-flow.md) — what the winning declarations actually compute
- [../05-dom-and-browser-event-loop.md](./05-dom-and-browser-event-loop.md) — inline styles and `classList` mutation from JS
- [../07-css-frameworks-bootstrap-tailwindcss.md](./07-css-frameworks-bootstrap-tailwindcss.md) — frameworks as a pre-made cascade
- [../08-forms-fetch-and-multipart-upload.md](./08-forms-fetch-and-multipart-upload.md) — form controls carry strong UA styles, so styling them fights the cascade
- [SUGGESTED] `bridge-rendering-pipeline.md` — where the CSSOM and cascade fit in the browser's render pipeline (HTML → DOM → CSSOM → layout → paint)
- [SUGGESTED] `09-css-animations-and-transitions.md` — animations and transitions occupy their own origin buckets at the top and bottom of the cascade
- [SUGGESTED] `10-css-naming-conventions-bem-itcss.md` — structural conventions that keep specificity low without `!important`

## 🤔 Open Questions
- [ ] Does `@layer` interact cleanly with inline `style` attributes and JS-written inline styles, or does inline always sit above layers? (Per spec: inline wins among author-normal regardless of layer.)
- [ ] What is the exact behaviour of `!important` inside `:where()` and `:is()` — does the argument's importance still apply even though the specificity is zeroed?
- [ ] How do shadow DOM (web components) encapsulation boundaries change cascade resolution versus a plain page?
- [ ] How much does a global reset/Preflight cost in terms of selector matching performance on very large DOMs?

---
*The cascade is not an obstacle to fight; it is the contract that lets thousands of declarations coexist.*
