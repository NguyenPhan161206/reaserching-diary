# 02 — Android App Fundamental Components

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-11
**Tags:** `#Android`, `#ActivityLifecycle`, `#MVC`, `#Intent`, `#Widgets`, `#Figma`, `#Gradle`
**References:** Android Developers — "Application Fundamentals", "The activity lifecycle", "Processes and app lifecycle"; `developer.android.com/guide/components`; Android Studio project docs; Material Design guidelines
**Status:** Draft
**Difficulty:** Beginner → Intermediate
**Prerequisites:** [`01-mobile-os-platforms-and-tools-overview.md`](01-mobile-os-platforms-and-tools-overview.md)

---

## 📌 Key Takeaways (TL;DR)
- An Android app is **not one program**. It is a bundle of **components** (Activity, Service, BroadcastReceiver, ContentProvider) that the *system* starts, stops, and destroys at will. Your job is to survive that.
- The **manifest** is the app's contract with the OS: it declares components, permissions, and hardware requirements. Nothing runs that isn't declared.
- **Lifecycle is a state machine with a lifecycle *outside* your control.** Every callback answers "what is the OS doing to me right now?" — configuration change, backgrounding, memory pressure, process death.
- **MVC is the mental model; the framework is not MVC out of the box.** Android's "View" (XML) is passive and your `Activity` is really a controller that got fused with a lifecycle host — the industry moved to MVVM to separate them (note 06 backlog).
- **Figma is the design source of truth**; the job is to translate its frames into a *view hierarchy*, not to eyeball pixels.

## 🧠 Mental Model
An Activity is a **page in a magazine that the reader (the OS) can turn, close, or throw away when low on paper**. You don't control the turning. You only implement the seams:

```
        ┌──────────┐  launch   ┌──────────┐
        │  (none)  │──────────►│ onCreate │
        └──────────┘           └────┬─────┘
                                    │
                              ┌─────▼──────┐        ┌──────────────┐
                              │ onStart    │───────►│ onResume     │  ← visible & interactive
                              └─────┬──────┘        └──────┬───────┘
                                    │  user leaves/rotate   │
                              ┌─────▼──────┐        ┌──────▼───────┐
                              │ onPause    │◄───────│  (running)   │
                              └─────┬──────┘        └──────────────┘
                              ┌─────▼──────┐
                              │ onStop     │  ← no longer visible
                              └─────┬──────┘
                         return   │        │ OS kills process
                              ┌─────▼─────┐  ┌──────────────┐
                              │ onDestroy │  │ process dies │  ← onCreate again if relaunched
                              └───────────┘  └──────────────┘
```

> **Why this matters:** the difference between `onStop` and `onDestroy`, and between "the Activity was destroyed" and "the *process* was killed", decides whether your app resumes gracefully or loses the user's typing. Every crash-on-rotation bug is a failure to internalize this diagram.

## 🔬 Detailed Notes

### 1. Android application architecture (or: what "an app" actually is)
At build time your app becomes an **APK/AAB** containing:
```
app/
├── AndroidManifest.xml     ← the contract: components, permissions, intent filters
├── classes.dex             ← compiled Kotlin → DEX bytecode
├── res/                    ← resources: layouts, strings, drawables, themes
└── META-INF/               ← signatures & metadata
```
At **runtime**, the OS (`ActivityManager`, `PackageManager`, `Binder`) is the conductor. It reads the manifest, knows which components exist, and launches them — possibly *without your app's process even running yet*. This is why an app can be started by a notification, a deep link, or another app's Intent, and why **cold start** (app process not alive) is the slowest path.

### 2. Project structure & the Gradle build system (Template G half)
```
MyApp/
├── settings.gradle.kts         ← which modules are in the build
├── build.gradle.kts (root)     ← top-level, plugin versions
├── gradle/libs.versions.toml   ← version catalog (modern dependency management)
└── app/
    ├── build.gradle.kts        ← module: dependencies, minSdk/targetSdk, build types
    ├── src/main/
    │   ├── AndroidManifest.xml
    │   ├── java/…/MainActivity.kt
    │   └── res/
    │       ├── layout/activity_main.xml
    │       ├── values/strings.xml, colors.xml, themes.xml
    │       └── drawable/, mipmap/
    └── src/test/, src/androidTest/   ← unit + instrumented tests
```
**Mental model:** `res/` is *resources*, addressed by an auto-generated `R` class (`R.layout.activity_main`, `R.string.app_name`). You never hardcode a string in the layout — you reference the resource, so the OS can pick the right translation/theme per device.

**Key `build.gradle.kts` knobs:**
```kotlin
android {
    namespace = "com.example.myapp"
    compileSdk = 35              // which SDK you compile against (can use new APIs)
    defaultConfig {
        minSdk = 24              // lowest device you support
        targetSdk = 35           // the behavior contract you've opted into
        versionCode = 1          // integer, must increase each release
        versionName = "1.0.0"    // human-readable
    }
    buildFeatures { viewBinding = true }   // type-safe view references (no findViewById)
}
```
> **`minSdk` vs `targetSdk` vs `compileSdk` is a classic exam/interview trap.** `minSdk` = the oldest device allowed. `targetSdk` = "I have tested against these behavior changes" (the OS enables compatibility shims below it). `compileSdk` = the API surface at compile time. Raising `targetSdk` is how Google *forces* you to adopt privacy/background changes.

### 3. The manifest — the app's contract
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Hardware/permission requirements — filtered by the Play Store -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="false" />

    <application
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp"
        android:icon="@mipmap/ic_launcher">
        <!-- The OS knows this component exists ONLY because it's declared here -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
`android:exported="true"` means *other apps may launch this component* — a security decision since Android 12 (undeclared means no external access). The `LAUNCHER` intent-filter is what puts an icon in the app drawer.

### 4. The four core components
| Component | Runs on | Purpose | Started by |
|-----------|---------|---------|-----------|
| **Activity** | Main thread | A single screen with UI | `startActivity(Intent)` |
| **Service** | Main thread (unless told otherwise) | Background work without UI | `startService` / `bindService` |
| **BroadcastReceiver** | Main thread, short-lived | React to system/app-wide events | `sendBroadcast(Intent)` |
| **ContentProvider** | Binder thread pool | Share structured data across apps | `ContentResolver.query()` |

> **The one rule that explains 90% of Android bugs: none of these run on a background thread by default.** An Activity, Service, and Receiver all execute their callbacks on the **main (UI) thread**. Doing network or DB work there freezes the UI (and throws `NetworkOnMainThreadException`). That is the entire reason note 05 (multithreading) exists.

**Activity — a screen:**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)          // always first
        setContentView(R.layout.activity_main)      // inflate the layout
        val button = findViewById<Button>(R.id.btnGreet)
        val input  = findViewById<EditText>(R.id.etName)
        button.setOnClickListener {
            Toast.makeText(this, "Hi ${input.text}", Toast.LENGTH_SHORT).show()
        }
    }
}
```
**Service** — long-running work with no UI (music playback, sync). Now strongly preferred: **foreground service** (shows a notification) for user-aware work, **WorkManager** for deferrable work (note 06).
**BroadcastReceiver** — e.g., react to `ACTION_BOOT_COMPLETED` or `CONNECTIVITY_CHANGE`. Since Android 8, most implicit broadcasts can't be registered in the manifest; use a *runtime* receiver or WorkManager.
**ContentProvider** — how the Contacts/Media apps expose their data to you via `ContentResolver` and URIs like `content://media/external/images/media/123`.

### 5. Activity lifecycle, back stack, and state
**The callbacks and their contract:**
| Callback | Means | Do this | Don't do this |
|----------|-------|---------|---------------|
| `onCreate` | "Create me" | Inflate UI, bind data, restore `Bundle` | Blocking work |
| `onStart` | "Become visible" | Register listeners | — |
| `onResume` | "Become interactive (foreground)" | Start animations, acquire resources | — |
| `onPause` | "Lose focus (partial cover)" | Stop animations, commit unsaved edits | Heavy work (blocks next Activity) |
| `onStop` | "No longer visible" | Release camera/sensors, save durable state | — |
| `onDestroy` | "I'm being destroyed" | Clean up, cancel callbacks | Assume app is exiting |
| `onRestart` | "Coming back after onStop" | Re-init what onStop released | — |

**Two distinct destructions (a subtle but critical distinction):**
- **Instance destroyed (e.g., rotation):** `onDestroy` fires, but the OS will recreate the Activity — `onSaveInstanceState(Bundle)` preserves small UI state.
- **Process killed (low memory, backgrounded):** no callback guaranteed; the whole process may die and be recreated later from a *saved* Bundle. This is why you must persist user-entered data at `onPause`/`onStop`, not just `onSaveInstanceState`.

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("draft", binding.etNote.text.toString())  // survives rotation
}
```

**The back stack** is a stack of Activities the OS maintains. `startActivity(B)` pushes B on top of A; Back pops B and resumes A. `finish()` pops voluntarily. This model predates the newer **single-Activity + Fragments + Navigation Component** style (note 03) — know both.

**Configuration changes:** rotation, keyboard, locale, dark mode → by default the Activity is destroyed and recreated (so `onCreate` runs again). Handle transient UI state with `onSaveInstanceState` or, better, a retained **ViewModel** (note 06 backlog).

### 6. MVC in Android — and why the industry outgrew it
Model-View-Controller, mapped onto Android:
- **Model** — your data (Room entities, networking DTOs, business logic).
- **View** — the XML layout + widget tree (passive; renders, reports events).
- **Controller** — in classic Android, the **Activity/Fragment** receives input and coordinates model↔view.

**Problem:** the Activity is *also* the lifecycle host. So the Controller ends up mixing lifecycle handling + UI logic + data access → the famous "God Activity" (thousands of lines). The industry response:
```
MVC   → Activity is controller AND view host (fused)
MVVM  → ViewModel holds UI state, Views observe it (ViewModel survives rotation)
MVI   → single immutable UI state, unidirectional data flow
```
This note teaches MVC because the course requires it; the backlog note on MVVM is where a modern app actually lives.

### 7. Figma → code: the design translator
**Figma mental model:** a *frame* is a screen; *auto-layout* is a flex container; components/variants are reusable widgets; constraints/auto-layout define how things resize. Your job is to map:
| Figma concept | Android concept |
|---------------|-----------------|
| Frame | `ConstraintLayout` / a screen |
| Auto layout (vertical/horizontal + gap) | `LinearLayout` / `ConstraintLayout` chain + `margin` |
| Component / variant | A reusable custom View or include/merge layout |
| Text style | `style` / `TypeAppearance` in `themes.xml` |
| Color style | `colors.xml` + theme attribute |
| Spacing tokens | `dimens.xml` |
| Prototype link (tap → next frame) | `Intent` / Navigation action |

**Linking design to code (the concrete bridge):** every element with an `id` in Figma gets an `android:id="@+id/..."` in XML, referenced in Kotlin via ViewBinding:
```kotlin
private lateinit var binding: ActivityMainBinding
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)
    binding.btnGreet.text = getString(R.string.greet)   // no findViewById, type-safe
}
```

### 8. Basic UI widgets
| Widget | Shows | Key attributes / notes |
|--------|-------|------------------------|
| `TextView` | Read-only text | `textSize`, `textColor`, `maxLines` |
| `EditText` | Editable text | `inputType` (email/number/password), `hint` |
| `Button` | Tap action | `text`; listener via `setOnClickListener` |
| `ImageButton` | Icon tap (no text) | `srcCompat`, needs `contentDescription` (a11y) |
| `ImageView` | Image | `scaleType` (centerCrop/fitCenter) |
| `CheckBox` | Independent multi-select | `isChecked`; combine with `OnCheckedChangeListener` |
| `RadioButton` + `RadioGroup` | Single-select | RadioButtons must be *inside* a RadioGroup to be exclusive |
| `Spinner` | Dropdown pick (deprecated-ish) | Backed by an `ArrayAdapter`; consider Material Exposed Dropdown |
| `ProgressBar` | Indeterminate spinner or determinate bar | `style="?android:attr/progressBarStyleHorizontal"` |
| `Picker` (DatePicker/TimePicker) | Date/time selection | Wrap in a `DialogFragment` (note 03) |

```xml
<!-- res/layout/activity_form.xml (excerpt) -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <EditText android:id="@+id/etEmail"
        android:inputType="textEmailAddress"
        android:hint="@string/email_hint"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button android:id="@+id/btnSubmit"
        android:text="@string/submit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <ProgressBar android:id="@+id/progress"
        android:visibility="gone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

## ⚖️ Trade-offs Summary
| Decision | Option A | Option B |
|----------|----------|----------|
| UI state on rotation | `onSaveInstanceState(Bundle)` (small, Parcelable) | ViewModel (survives rotation, not process death) |
| Navigation model | Multi-Activity + back stack | Single-Activity + Fragments + Navigation (recommended now) |
| Code structure | Activity does everything (fast to start) | MVC/MVVM separation (scales, testable) |
| Storing a form draft | In-memory field | Persist at `onStop` (survives process death) |

## 🏭 Industry Corners & Connected Tech
- **Lifecycle is the mobile analogue of the web request lifecycle** — in Express the server controls the request/response; here the *OS* controls when your component runs. Compare [`../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md).
- **MVC→MVVM** mirrors the same evolution the web had (server-rendered controllers → client-side state stores). The problem "who owns state?" is universal.
- **Figma ↔ engineering workflow:** design tokens (`dimens.xml`, color styles) are the mobile twin of a design system in web CSS.
- **Accessibility (a11y):** `contentDescription`, touch target sizes, and screen readers are legal requirements in many markets — a design-and-code concern, not a nice-to-have.

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Doing I/O in `onCreate` | ANR / frozen UI | Move to coroutine/background (note 05) |
| Assuming `onDestroy` == app exit | State lost, resources not released | Treat every callback as "maybe the last" |
| Forgetting `onSaveInstanceState` | Typed text vanishes on rotation | Save small state; use ViewModel for real state |
| Missing `android:exported` | Install/build failure on API 31+ | Declare explicitly |
| Hardcoded strings | No i18n, lint warnings | `strings.xml` + `getString()` |
| `findViewById` everywhere | `NullPointerException`, brittle | ViewBinding |
| Radio buttons not in a RadioGroup | Multiple can be selected | Wrap in `RadioGroup` |

## 🧪 Practice Tasks
- [ ] **Task 1** — Build a two-Activity app (Form → Result). Type text in A, rotate the device, confirm the text survives (`onSaveInstanceState` + restoration). Log every lifecycle callback with a tag and screenshot the sequence.
- [ ] **Task 2** — Convert Task 1 to use ViewBinding and a `dimens.xml`/`strings.xml` with zero hardcoded values. Run Android Lint; make it clean.
- [ ] **Task 3** (harder) — Reproduce the "God Activity" smell on purpose: put DB access directly in `onCreate`. Explain (from the lifecycle contract) exactly why this is wrong, then name which modern component fixes it.

## 📝 Research Journey
- **Why:** The lifecycle diagram was the first thing that made Android feel *principled* rather than random — I wanted the underlying contract, not a tutorial to copy.
- **Struggle:** Separating "instance destroyed" from "process killed" took real effort; tutorials gloss over it, and yet it's the root of most state-loss bugs.
- **Aha moment:** Realizing the manifest is literally a *contract* ("these components exist and here's who may call them") collapsed several confusing behaviors (why `am start` works, why exported matters, why a notification can wake a dead app) into one idea.
- **Career link:** System-design interviews reward exactly this "the platform owns the execution" reasoning; it's the same skill as reasoning about serverless cold starts.

## 🔗 Related Topics
- [`01-mobile-os-platforms-and-tools-overview.md`](01-mobile-os-platforms-and-tools-overview.md) — the platform/toolchain this builds on
- [`03-user-interface-programming.md`](03-user-interface-programming.md) — layouts, navigation, fragments in depth
- [`05-multithreading.md`](05-multithreading.md) — why the main thread must never block
- [`../web_server/03-mvc-architecture-in-express.md`](../web_server/03-mvc-architecture-in-express.md) — MVC in a different runtime
- [SUGGESTED] `06-jetpack-viewmodel-and-mvvm.md` — the modern answer to the lifecycle/state problems raised here

## 🤔 Open Questions
- [ ] At what app size does the MVC→MVVM migration become mandatory rather than nice?
- [ ] With Compose, does "Lifecycle" change meaning, or just the way we observe it?
- [ ] How should `onSaveInstanceState` size limits shape what we choose to persist vs recompute?

---
*The OS will start, stop, and kill you at will. Your only real job is to leave clean seams.*
