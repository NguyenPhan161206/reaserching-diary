# 03 — User Interface Programming

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-11
**Tags:** `#Android`, `#UI`, `#Layouts`, `#RecyclerView`, `#Fragments`, `#Navigation`, `#WebView`, `#Dialogs`
**References:** Android Developers — "Layouts", "Navigation", "RecyclerView", "Fragments", "Menus", "Dialogs", "WebView"; Material Design 3 guidelines; ConstraintLayout docs
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md)

---

## 📌 Key Takeaways (TL;DR)
- The UI is a **tree of Views**. Everything else — layouts, lists, fragments, dialogs — is arranging that tree efficiently.
- A **layout manager is a constraint solver, not a position list.** `ConstraintLayout` lets you describe relationships ("B is below A, centered between margins") and solves for the pixels, which is why it survives every screen size.
- **`RecyclerView` exists because views are expensive.** It *recycles* the small number of row views that fit on screen instead of creating one per data item — the difference between a smooth list of 100,000 rows and an OOM crash.
- **Fragments are reusable, replaceable slices of a screen**, not mini-Activities. Modern apps use **one Activity + many Fragments + the Navigation Component**.
- Dialogs, menus, and the WebView are all **components the OS owns**; you request them and hand back control.

## 🧠 Mental Model
A screen is a **tree**, and each node measures itself then places its children:

```
        DecorView (the window)
            │
        ConstraintLayout                 ← root layout
        ┌──────┴───────────────┐
     ImageView             LinearLayout (vertical)
                            ┌──────┴──────┐
                        TextView      RecyclerView
                                      ┌───┴────┐
                                    row0     row1  ← recycled as you scroll
```

**The two-pass layout algorithm** (the single most useful thing to know for UI bugs):
1. **Measure pass** — parent hands each child a size *spec* (`AT_MOST`, `EXACTLY`, `UNSPECIFIED`); child reports its desired size.
2. **Layout pass** — parent assigns final bounds; then draw.

Nested `LinearLayout`s with `layout_weight` force repeated measurement (each weight requires re-measuring siblings). That is *why* deep view trees jank and why `ConstraintLayout` (a single flat solver) was invented.

## 🔬 Detailed Notes

### 1. Layouts & the view hierarchy
| Layout | Model | Use when | Avoid when |
|--------|-------|----------|-----------|
| `FrameLayout` | Stack (z-order) | Overlays, a single child | Aligning many children |
| `LinearLayout` | Row/column, optional weight | Simple lists/rows | Deep nesting, weight-heavy |
| `RelativeLayout` | Relative to siblings | Legacy code | New code (use ConstraintLayout) |
| `ConstraintLayout` | **Constraint solver** (flat) | Almost everything; complex responsive UI | Trivial single-child (overkill) |
| `GridLayout` | Grid cells | Small fixed grids | Long scrolling grids (use a RecyclerView+GridLayoutManager) |

**ConstraintLayout essentials:**
```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView android:id="@+id/avatar"
        android:layout_width="64dp"
        android:layout_height="64dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_margin="16dp"/>

    <TextView android:id="@+id/name"
        android:layout_width="0dp"          <!-- 0dp = "match constraints" -->
        android:layout_height="wrap_content"
        app:layout_constraintStart_toEndOf="@id/avatar"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="@id/avatar"
        android:layout_marginStart="12dp"/>

    <Button android:id="@+id/follow"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toBottomOf="@id/name"
        app:layout_constraintStart_toStartOf="@id/name"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```
`layout_width="0dp"` + both start/end constraints = "fill the space between my anchors". Chains (`app:layout_constraintHorizontal_chainStyle`), barriers, and guidelines handle responsive alignment.

### 2. Simple vs composite views
- **Simple view:** a single widget (`TextView`, `Button`) — one node.
- **Composite view:** a group of views treated as one reusable unit. Build one by:
  1. `include`-ing a layout, or
  2. a **custom View / ViewGroup subclass**, or
  3. a **layout with `<merge>`** to cut a redundant wrapper.

```xml
<!-- res/layout/view_profile_header.xml -->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <ImageView android:id="@+id/hAvatar" ... />
    <TextView  android:id="@+id/hName"   ... />
</merge>
```
> The `<merge>` trick matters: `<include>` of a normal layout adds one extra `ViewGroup` level; `<merge>` flattens it into the parent, reducing measure/layout cost — the same instinct as avoiding wrapper `<div>`s in web layout.

### 3. Multi-screen navigation & passing parameters
**Two models, and the industry switched:**
```
OLD:  Activity A ──Intent──► Activity B ──Intent──► Activity C   (back stack of Activities)
NEW:  MainActivity ──Fragment──► Fragment ──► Fragment            (one Activity, Nav Component)
```

**Intent-based (classic):**
```kotlin
// Start a new screen, passing typed extras
val intent = Intent(this, DetailActivity::class.java).apply {
    putExtra("EXTRA_ID", item.id)
    putExtra("EXTRA_NAME", item.name)
}
startActivity(intent)

// On DetailActivity
val id = intent.getStringExtra("EXTRA_ID")
```
**Make navigation safe and typed with `SafeArgs`** (Navigation Component plugin):
```kotlin
// Generated from nav_graph.xml → no string keys, no casts, compile-time checked
val action = ListFragmentDirections.actionListToDetail(item.id)
findNavController().navigate(action)
```
**Passing-data decision table:**
| Need | Mechanism |
|------|-----------|
| Small primitive (id, name) | Intent extra / SafeArgs arg |
| Complex object | Pass the **id**, reload from DB/source (avoid giant Parcelables) |
| Return a result from a screen | Activity Result API (`registerForActivityResult`) |
| Deep link (open a screen from a URL/notification) | Intent-filter + Nav deep links |

**Deep links** are why an app can open "straight to the order screen" from an email — a URI maps to a destination. This is the mobile twin of web routing ([`../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md)).

### 4. Lists — `RecyclerView`
The mental model: **the view is a moving window over the data.**
```
data items:  [0][1][2][3][4][5][6][7]...      (100,000 items)
on-screen:           ┌─────┐
                     │2│3│4│                    ← only ~N views exist
scroll down →        └─────┘
                     │3│4│5│                    ← view "2" is recycled to show "5"
```
Three pieces:
```kotlin
class ItemAdapter(private val items: List<Item>) :
    RecyclerView.Adapter<ItemAdapter.VH>() {

    class VH(view: View) : RecyclerView.ViewHolder(view) {
        val title: TextView = view.findViewById(R.id.tvTitle)
        val subtitle: TextView = view.findViewById(R.id.tvSubtitle)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): VH {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.row_item, parent, false)   // created only when a NEW row enters
        return VH(view)
    }

    override fun onBindViewHolder(holder: VH, position: Int) {
        val item = items[position]
        holder.title.text = item.title            // reuse an existing view's slots
        holder.subtitle.text = item.subtitle
    }

    override fun getItemCount() = items.size
}
```
**Optimization toolkit (this is where UI performance lives):**
- **`ViewHolder` pattern** — cache `findViewById` results (mandatory).
- **`DiffUtil` / `ListAdapter`** — compute the minimal change set so you redraw only what changed (instead of `notifyDataSetChanged()` which rebinds everything).
- **Pagination / `Paging` library** — load pages as the user scrolls (ties into note 05's network work).
- **Avoid `onBindViewHolder` work** — no DB queries, no image decoding in binding; use Glide/Coil (async image loading).

> **Industry corner:** a list is the mobile twin of a web virtual-list / windowing. Same idea (render only what's visible), same trade-off (complexity vs performance).

### 5. Fragments
A **Fragment** is a reusable piece of UI with its own lifecycle, hosted by an Activity. Use it for: split screens (tablet master/detail), tabs, reusable panels, and the single-Activity navigation model.
```kotlin
class DetailFragment : Fragment(R.layout.fragment_detail) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val id = arguments?.getString("id")
        // bind, observe, etc.
    }
}
```
**Fragment lifecycle is longer than the Activity's** and includes `onAttach`/`onCreateView`/`onViewCreated`/`onDestroyView`. The classic bug: holding a reference to a View after `onDestroyView` → **memory leak**. Get the fragment's own lifecycle right and the Activity's follows.

**Single-Activity + Navigation Component (the modern default):**
```
MainActivity
   └── NavHostFragment  ← hosts a nav graph
          ├── HomeFragment
          ├── ListFragment
          └── DetailFragment
```
Navigation is declared in `nav_graph.xml`; the back stack, animations, arguments, and deep links are managed for you.

### 6. ActionBar & menus
| Menu type | Where | Created in |
|-----------|-------|-----------|
| Options menu | App bar (overflow ⋮) | `onCreateOptionsMenu` / `onOptionsItemSelected` |
| Context menu | Long-press an item | `registerForContextMenu` |
| Popup menu | Anchored to a view | `PopupMenu(view).show()` |

```kotlin
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    menuInflater.inflate(R.menu.main_menu, menu)
    return true
}
override fun onOptionsItemSelected(item: MenuItem): Boolean = when (item.itemId) {
    R.id.action_settings -> { openSettings(); true }
    R.id.action_search   -> { openSearch(); true }
    else -> super.onOptionsItemSelected(item)
}
```
Modern apps use the **Material Toolbar** (a customizable AppBar) instead of the platform ActionBar; the menu API is the same.

### 7. Dialogs
| Dialog | Purpose |
|--------|---------|
| `AlertDialog` | Confirm/notify with buttons |
| `DatePickerDialog` / `TimePickerDialog` | Pick a date/time |
| `BottomSheetDialog` | Material bottom sheet |
| `DialogFragment` | **The recommended host** for any dialog (survives rotation, lifecycle-safe) |

```kotlin
AlertDialog.Builder(requireContext())
    .setTitle("Delete?")
    .setMessage("This cannot be undone.")
    .setPositiveButton("Delete") { _, _ -> doDelete() }
    .setNegativeButton("Cancel", null)
    .show()
```
> **Never** show a raw `AlertDialog` from a background thread or after the Activity may be gone — wrap it in a `DialogFragment` that the FragmentManager manages, which also survives configuration changes.

### 8. WebView — embedding web content
`WebView` is a mini-browser inside your app (Chrome's rendering engine on modern Android). Use it for: content you already have as web, terms/help pages, or hybrid apps. Do **not** use it as "the whole app" unless that's the deliberate strategy (Ionic/Capacitor).
```kotlin
val web = findViewById<WebView>(R.id.webView)
web.settings.javaScriptEnabled = true            // enable only if you trust the content
web.webViewClient = WebViewClient()              // keep navigation INSIDE the WebView
web.loadUrl("https://example.com")

// Talk between Kotlin and JS
web.addJavascriptInterface(WebAppInterface(), "Android")
```
**Security corner (critical):** `addJavascriptInterface` exposes native methods to JS — if the page is untrusted, that's a remote-code-execution vector. Prefer `WebViewAssetLoader` for local content, `shouldOverrideUrlLoading` to whitelist domains, and never enable JS for untrusted URLs. This is the mobile cousin of XSS ([`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)).

## ⚖️ Trade-offs Summary
| Decision | Option A | Option B |
|----------|----------|----------|
| Navigation | Multi-Activity (simple, classic) | Single-Activity + Navigation (modern, deep-link friendly) |
| Layout | Deep nested LinearLayouts (intuitive) | Flat ConstraintLayout (faster, harder to read) |
| List updates | `notifyDataSetChanged()` (easy) | `DiffUtil`/`ListAdapter` (efficient, more code) |
| Dialog | inline `AlertDialog` (quick) | `DialogFragment` (lifecycle-safe) |
| WebView content | Remote URL (flexible, untrusted) | Local assets via `WebViewAssetLoader` (safe) |

## 🏭 Industry Corners & Connected Tech
- **Layout performance ↔ the web rendering pipeline:** measure/layout/draw maps to the browser's reflow/repaint. Avoiding deep nesting is the same lesson as avoiding layout thrash in CSS.
- **RecyclerView ↔ virtual lists in React/Next.js:** identical windowing insight, different runtime. See [`../web_basics/05-dom-and-browser-event-loop.md`](../web_basics/05-dom-and-browser-event-loop.md).
- **Navigation/deep links ↔ server routing:** a URI maps to a destination. Compare [`../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md).
- **Accessibility:** touch-target size, contrast, TalkBack labels — a legal requirement in many markets and a design-system concern.
- **Material Design ↔ psychology:** elevation, motion, and hierarchy are perception engineering; see [`../../psychology_society/README.md`](../../psychology_society/README.md).

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Building a View per list item | Jank / OOM on long lists | RecyclerView + ViewHolder |
| Heavy work in `onBindViewHolder` | Scroll stutter | Move I/O & decoding off bind (Coil/Glide, coroutines) |
| Holding a View ref in a Fragment | Memory leak | Null refs at `onDestroyView` |
| Raw `AlertDialog` after rotation | `IllegalStateException` / leak | `DialogFragment` |
| JS enabled for untrusted WebView | RCE / data theft | Whitelist domains; disable JS; `WebViewAssetLoader` |
| Passing huge objects via Intent | `TransactionTooLargeException` | Pass an id, reload from source |

## 🧪 Practice Tasks
- [ ] **Task 1** — Build a 3-screen app with the Navigation Component: List → Detail → Settings. Pass an item id to Detail via SafeArgs. Verify Back and Up behave correctly, and that rotation does not lose the Detail screen.
- [ ] **Task 2** — Implement a `RecyclerView` of 10,000 rows with `ListAdapter` + `DiffUtil`. Change one item and confirm (via logs or animation) that only that row rebinds.
- [ ] **Task 3** (harder) — Embed a WebView that loads a **local** HTML file via `WebViewAssetLoader`, with JS disabled, then make a button in the HTML call a whitelisted Kotlin method. Explain the security boundary you just drew.

## 📝 Research Journey
- **Why:** UI is where mobile is *felt*; I wanted to understand not just "how to place a button" but why the framework is shaped the way it is (measure/layout, recycling).
- **Struggle:** The RecyclerView recycling model was abstract until I logged `onCreateViewHolder` vs `onBindViewHolder` and watched creation happen only as new rows scrolled in.
- **Aha moment:** Seeing that `ConstraintLayout` is literally a constraint *solver* reframed layout from "positions" to "relationships" — which is the same reframe as CSS Flexbox/Grid vs absolute positioning.
- **Career link:** performance-sensitive UI reasoning (windowing, avoiding redundant work) is the same skill as optimizing a data pipeline — measure, find the repeated work, eliminate it.

## 🔗 Related Topics
- [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md) — views, widgets, MVC, lifecycle
- [`05-multithreading.md`](05-multithreading.md) — keeping the UI thread free while lists load data
- [`../web_basics/02-css-box-model-and-layout-flow.md`](../web_basics/02-css-box-model-and-layout-flow.md) — layout as a constraint/flow problem
- [`../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md) — routing/deep links as navigation
- [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md) — WebView security parallels
- [SUGGESTED] `bridge-xml-views-to-compose.md` — the declarative-UI rewrite of everything here

## 🤔 Open Questions
- [ ] How does Compose change the RecyclerView/recycling story — is manual windowing still a developer concern?
- [ ] Where is the line between a custom View and a composite layout for maintainability?
- [ ] Can deep links be modeled as a route table the same way a web server does, for a single source of truth?

---
*The UI is a tree, the OS measures it, and recycling is how you afford to show infinity.*
