# 01 — Mobile OS, Platforms & Tools Overview

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-12
**Tags:** `#Android`, `#iOS`, `#MobileOS`, `#CrossPlatform`, `#Toolchain`, `#AppStore`
**References:** Android Developers docs (`developer.android.com`), Apple Developer docs, Google Play Policy Center, AOSP source, Flutter/Xamarin/.NET MAUI/React Native official docs, `Gartner`/`StatCounter` mobile OS share
**Status:** Draft
**Difficulty:** Beginner
**Prerequisites:** [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md), [`../../git_github/README.md`](../../git_github/README.md)

---

## 📌 Key Takeaways (TL;DR)
- A phone is **not a small PC**. Four physical constraints — battery, thermal budget, intermittent network, and a hostile input surface — decide almost every mobile architecture choice before a line of code is written.
- **Android is a Linux kernel fork**, not "Linux with an app store". Binder IPC, the ART runtime, and the permission model are what make it *Android* (note 02).
- **Native ≠ better, cross-platform ≠ faster to ship.** The right choice is a function of team skills, platform features needed, and how much UI must feel "at home" — not a religious preference.
- The toolchain is a **pipeline**: source → Gradle → D8/R8 → APK/AAB → signed → Play Console review → user. Knowing where a failure lives in that pipeline *is* debugging.
- An **app store is a gatekeeper with a policy**, not just a file host. Most first-app rejections are policy/metadata issues, not code bugs.

## 🧠 Mental Model
Think of the mobile stack as **rings around a battery-limited computer**:

```
        ┌─────────────────────────────────────────────────────┐
        │  App (your Kotlin code, UI, logic)                  │
        ├─────────────────────────────────────────────────────┤
        │  Framework layer: Activity, View, Room, Retrofit    │  ← what you import
        ├─────────────────────────────────────────────────────┤
        │  Runtime: ART (Android) / ObjC-Swift (iOS)          │
        ├─────────────────────────────────────────────────────┤
        │  System services: Binder, PackageManager, sensors   │
        ├─────────────────────────────────────────────────────┤
        │  Kernel: Linux (Android) / XNU (iOS)                │
        ├─────────────────────────────────────────────────────┤
        │  Hardware: SoC, radio, battery, sensors             │  ← the real boss
        └─────────────────────────────────────────────────────┘
```

Every layer can **kill your process** to reclaim memory, and every layer above the hardware exists because the layer below it is limited. When something behaves "weirdly" on mobile (app restarts, background work stops, a toast appears late), the cause is almost always a *lower ring policing the upper one* — the OS reaping your process, a Doze window batching your alarm, the radio waking from sleep.

> **Why this matters:** a desktop developer assumes their program owns the machine. A mobile developer must assume the machine owns *them*. This single mindset shift is the difference between code that ships and code that ANRs.

**The shift, stated as a table** — because the diagram above is easy to nod at and hard to actually believe until it is written out:

| The developer assumes… | On desktop | On mobile |
|------------------------|------------|-----------|
| **Ownership** | My process owns the machine until *I* exit | The OS owns the machine and may kill *me* at any moment to reclaim memory |
| **Time** | A background job can run for minutes | Background work is *scheduled*, batched, and can be deferred past a Doze window |
| **Resources** | RAM is cheap; a leak is merely wasteful | RAM is shared with every other app; a leak gets my process reaped |
| **Connectivity** | The network is normally present | The network is an *exception*; offline is the default state |
| **Input** | Precise pointer, hover, right-click, keyboard | Fat finger, no hover, one hand, direct sunlight |

> **The deeper consequence:** the four "weird" mobile behaviours (an app restarting, background work stopping, a late notification, a dropped frame) are not bugs to work around — they are the **OS enforcing the constraints above**. When you meet one, ask *which ring in the diagram is policing me*, not *how do I defeat it*.

## 🔬 Detailed Notes

### 1. Evolution of the phone — three eras
| Era | Hardware | Software consequence |
|-----|----------|----------------------|
| **Feature phone (pre-2007)** | Fixed keypad, tiny OS (Symbian, J2ME, BREW) | Apps are tiny, carrier-controlled, no app store |
| **Early smartphone (2007–2012)** | Multi-touch, single core, App Store + Play launched | Third-party apps become a market; sandboxing + permissions appear |
| **Modern phone (2012–now)** | 8-core SoC, NPU, 5G, many cameras/sensors, foldables | Apps are clients to cloud services; on-device ML; privacy as an OS feature |

The iPhone (2007) and Android (2008) didn't just add a touchscreen — they introduced the **two ideas** the whole industry rests on: (1) a **sandboxed third-party app model**, and (2) an **app store as the distribution channel** (ergo the gatekeeper in §8).

### 2. Device characteristics & the four constraints
| Constraint | Physical cause | What it forces you to do |
|-----------|----------------|--------------------------|
| **Battery** | Finite mAh; radio is the biggest drain | Batch network calls; avoid wake locks; use WorkManager; respect Doze/App Standby |
| **Thermal** | Phones have no fan; sustained CPU/GPU throttle | Reduce sustained load; move heavy work off-device; expect FPS to drop under heat |
| **Intermittent network** | Mobile radio + tunnels + dead zones | **Offline-first**: local DB is the source of truth, network is a sync |
| **Hostile input** | Small touchscreen, fingers, one-handed use, sunlight | Large touch targets (≥48dp), contrast, no hover, no right-click |

**The four constraints, explained past the table** — because "batch your network calls" is a rule most people memorise without the *why*:

- **🔋 Battery — the fixed cost of waking the radio dwarfs the payload.** The energy goes into bringing the mobile radio out of its low-power state and the idle "tail" that follows each transmission, not into the bytes themselves. Two consequences fall out of that one fact: (1) **100 small requests scattered across a minute can cost more than one batched request** carrying the same total data, because the radio power-cycles 100 times; (2) "keep the radio asleep" beats "send less data". This is the mechanism behind batching, `WorkManager`, and respecting Doze/App Standby (note 06).
- **🌡️ Thermal — *sustained* power, not *peak* power, is the binding constraint.** A phone has no fan, so heat escapes only through its surface. A desktop CPU can burst to 100% for minutes and cool later; a phone throttles *within seconds* and then holds a lower clock while it is hot. The design consequence is counter-intuitive: an algorithm that finishes in a **3-second burst** can deliver better sustained performance than one that keeps every core busy for a minute, because the second one triggers throttling and pays for it. Optimise for **joules per task**, not cycles per task.
- **📶 Intermittent network — "offline-first" is a *data-ownership* decision, not a UI trick.** The concrete rule: the **local database is the source of truth**; the network is only a *replication channel*. Writes land locally first and sync later. That is why every serious mobile app has a local store (note 04) and a sync strategy (notes 05–06). A "network-first" app shows a blank screen the moment a tunnel swallows the signal.
- **🖐️ Hostile input — 48dp is a psychological constant, not a style choice.** `dp` is a *density-independent* unit, so 48dp is ≈ **9 mm** on every device regardless of pixel density — about a fingertip. Material's 48dp minimum touch target is a straight application of **Fitts's Law** (time to acquire a target shrinks as the target grows and the distance to it shrinks). Mobile UI is where engineering meets perceptual psychology → see [`../../psychology_society/README.md`](../../psychology_society/README.md).

**Device fragmentation** is the fifth, software constraint: thousands of device models, Android API levels spanning 10+ years, manufacturer skins (One UI, MIUI), and varying screen densities/fold states. A desktop app targets ~3 OS versions; an Android app may target **API 24 → 35+** and 5 screen size classes. This is why responsive layout (note 03) isn't optional.

### 3. Mobile OS: Android vs iOS
| Aspect | Android | iOS |
|--------|---------|-----|
| Kernel | Linux (GPL) fork + AOSP | XNU (Darwin/BSD + Mach) |
| Runtime | **ART** (AOT+JIT), Kotlin/Java → DEX | Native Swift/Obj-C compiled (Arm64) |
| App packaging | APK / **AAB** (Play splits per device) | IPA |
| Distribution | Play Store + sideload + third-party stores | App Store only (except EU sideloading via DMA, 2024) |
| Language (modern) | **Kotlin** (Java legacy) | **Swift** (Obj-C legacy) |
| UI toolkit | XML Views / Jetpack Compose | UIKit / SwiftUI |
| Isolation | Per-app Linux UID + sandbox + SELinux | Per-app sandbox + entitlements |
| Security posture | Open ecosystem, vendor patches vary | Closed, unified update path, stricter review |
| Store fee | 15–30% (Play) | 15–30% (App Store) |

The two OSes converge on the *concepts* (sandbox, permission prompts, background restrictions, declarative UI) and diverge on the *mechanisms*. Learn concepts once; the mechanisms are the per-platform tax.

**Why the two runtimes diverge — and what it actually costs you:**
- **Android — bytecode on a managed runtime.** Kotlin (or Java) compiles to **JVM bytecode**; `D8` rewrites that to **DEX**; **ART** runs it, ahead-of-time (AOT) compiling at install and JIT-compiling at runtime. Shipping bytecode means the *same* app runs on ARM and x86 without recompiling across thousands of device models, and you get a **garbage collector**, reflection, and dynamic class loading for free. The price is a VM layer between your code and the metal — GC pauses are real, and peak throughput is below bare native.
- **iOS — machine code on the metal.** Swift/Obj-C compile (via LLVM) straight to **native arm64 machine code**; there is no VM and no bytecode step at runtime. The upside is deterministic performance and no GC pauses; the cost is losing the portability abstraction — Apple controls the single hardware path and every rule.
- **The transferable lesson:** *sandbox, permission prompts, background kill-rules, and declarative UI* are now **concepts** both platforms share (Jetpack Compose on Android ↔ SwiftUI on iOS are the same idea from two vendors). Learn the concept once; the per-platform *mechanism* — Binder vs XPC, DEX vs Mach-O, ART vs no-runtime — is the tax you pay for each platform you ship on. This is exactly why a native-Android developer can pick up Flutter or React Native, while the reverse is not guaranteed.

### 4. App stores, accounts, and cloud resources
- **App store role:** discovery, distribution, payments, updates, and **policy enforcement**. It is a two-sided market — it earns the developer access to billions of devices in exchange for a fee and rules (see `economics_business/04-network-effects.md` for why this market concentrates so hard).
- **Developer account:** required to publish. Google charges a **one-time \$25**; Apple charges **\$99/year** (and Apple is stricter — e.g., requires a real legal entity name for some categories).
- **Why developers tolerate the gate (the two-sided view):** a store is not only a toll booth; it is **trust infrastructure**. Before curated stores, users would not install unknown apps (malware risk), so the *addressable market* for a small developer was tiny. A store that vets apps raises consumer willingness to install, which grows the very pie the developer sells into; the 15–30% fee is the price of that trust plus distribution. Whether that price is *fair* is a genuinely open question (see the EU DMA point in §6 and Industry Corners) — but framing it as pure extortion ignores the demand-side value the gate creates.
- **Cloud resources** the ecosystem hands you: push (FCM/APNs), crash reporting (Play Console/App Store Connect), analytics, and identity. These are why note 06's Firebase chapter exists — the platform *wants* you on its managed services.
- **The store's power is *policy*, not *physics*.** Google charges a one-time \$25, permits sideloading and third-party stores. Apple charges \$99/year, reviews every build, and until the EU's DMA (2024) forbade sideloading entirely. That gap is a business/policy decision, not a technical limit — which is precisely why regulators *can* force it to move.
- **Accounts you'll actually juggle:** Play Console (publish), Google Cloud (Firebase, Maps API keys), GitHub/GitLab (code), and the emulator's Google account (for Play Services dependencies). This cluster of accounts is also where API keys leak most often — keys belong in `local.properties`, never committed (see Pitfalls).

### 5. Data access permissions, security & MDM
Mobile security is built on **least privilege + explicit consent**:

**The permission model is a timeline, not a single design** — and the direction of travel is always *stricter*:

| Android version | Mechanism | The problem it solved |
|-----------------|-----------|-----------------------|
| ≤ 5.x (API <23) | **Install-time only** — one "permission wall" at install | Usability — but users approved blindly, so consent was nominal |
| 6.0 (API 23) | **Runtime permissions** for *dangerous* permissions | Consent at the moment of use; users deny only what they dislike |
| 10 (API 29) | **Scoped storage** | An app no longer sees the whole filesystem (privacy) |
| 11 (API 30) | **One-time** + **while-in-use** grants | Stops silent background access (e.g. location) |
| 13 (API 33) | Granular media (`READ_MEDIA_IMAGES/VIDEO/AUDIO`) | Replaces blanket `READ_EXTERNAL_STORAGE` |

Every change above targets a *silent access* an app used to get away with. The system-level lesson — the same one behind Linux file permissions (UID) and database RLS — is **least privilege**: grant the narrowest capability for the shortest time. Requesting permissions you do not need is now a *store-rejection* reason, not a neutral choice (see the Industry corner below).

- **Install-time / runtime permissions** (note 02 covers the mechanism): Android splits permissions into *normal* (auto-granted) and *dangerous* (prompt the user at runtime). Since Android 11, permissions can be **one-time** or **while-in-use only**; since 13, you request the specific media type (READ_MEDIA_IMAGES) instead of broad storage.
- **Scoped storage** (Android 10+): apps can't read the whole filesystem. Each app owns its private dir; shared storage access is mediated by MediaStore / the Storage Access Framework (note 04).
- **Storage encryption:** modern Android devices are encrypted by default; Keystore/Keychain holds keys. Passwords and tokens should never sit in plain `SharedPreferences`.
- **MDM (Mobile Device Management):** enterprise control plane — remote wipe, forced passcode, app allow-lists, work profiles (Android Work Profile / iOS Managed Device). This is how banks and hospitals deploy apps. A related concept is **MAM** (Mobile Application Management), which manages the app's data rather than the whole device.

> **Industry corner — privacy as a product:** GDPR (EU), CCPA (US), and platform rules (iOS App Tracking Transparency) turned "what data do you collect?" from an engineering afterthought into a **legal and store-review gate**. Requesting permissions you don't need is now a common rejection reason.

### 6. Native vs cross-platform frameworks
A **native** app is written in the platform's language against its SDK (Kotlin/Android, Swift/iOS). A **cross-platform** app writes once and renders/runs on both.

| Framework | Language | Rendering model | Best at | Main cost |
|-----------|----------|-----------------|---------|-----------|
| **Native Android** | Kotlin | Android Views/Compose | Full platform access, best perf/UX | iOS needs a second codebase |
| **Native iOS** | Swift | UIKit/SwiftUI | Full platform access, best iOS UX | Android needs a second codebase |
| **Flutter** | Dart | Own rendering engine (Skia/Impeller) | Pixel-perfect identical UI on both; great animation | Non-native look; Dart is a new language; heavier binary |
| **React Native** | JavaScript/TS | Bridges to native views (Fabric/new arch) | Web teams reuse JS+React skills | Bridge overhead; native modules when you need depth |
| **Xamarin / .NET MAUI** | C# | Maps to native controls | C#/.NET shops, enterprise | Smaller ecosystem momentum; MAUI is the successor to Xamarin.Forms |
| **Ionic/Capacitor** | Web (HTML/CSS/JS) | WebView | Web teams, content apps | WebView UX/perf ceiling |

**How to actually decide (multi-dimensional thinking):**
1. **Feature ceiling:** do you need heavy camera/Bluetooth/AR/background work? Native (or a cross-platform framework with a good native-escape hatch).
2. **UI fidelity:** must it feel native per platform (banking, system utilities)? Native/Flutter. Content app with brand UI? RN/web.
3. **Team skills:** your existing JS team → RN; C# team → MAUI; greenfield mobile team → Kotlin (native) or Flutter.
4. **Longevity:** who maintains the framework? (Xamarin→MAUI and the partial deprecation of some tools show framework risk.)
5. **Time-to-market vs technical debt:** cross-platform wins the first 3 months; native often wins year 3.

**Two hidden variables the table above does not show:**
- **The escape hatch is the real cost driver.** Every cross-platform framework eventually meets a feature it cannot express, and you drop to native code through its bridge — React Native *native modules*, Flutter *platform channels*, MAUI *handlers*. So the honest question is not *"can it do X?"* but **"how often will I need the escape hatch, and how expensive is one crossing?"** A content/brand-UI app crosses rarely and keeps the cross-platform advantage; a camera/Bluetooth/AR/background-heavy app crosses constantly and loses it entirely.
- **TCO flips over time.** Cross-platform wins *months 1–3* (one codebase, one team, fastest ship); native tends to win *year 3*. Framework churn is maintenance you did not budget for — Xamarin → MAUI, and React Native's Old vs New Architecture migration — and platform vendors always optimise their own first-class toolkit first. Pick by the horizon you actually need, and say it out loud.

> **This course's decision:** Android native with **Kotlin**. Why: it teaches the *platform concepts* directly (lifecycle, Binder, permissions) rather than hiding them behind a framework abstraction. A developer who understands native Android can pick up Flutter or RN; the reverse is not always true.

### 7. Native Android platform & native iOS platform (survey)
**Android platform anatomy** (detailed in note 02): Linux kernel → Hardware Abstraction Layer → native C/C++ libs (SQLite, OpenGL, libc `bionic`) → Android Runtime (ART) → Java/Kotlin framework APIs → your app. Apps ship as DEX bytecode run by ART; the manifest declares components; Binder is the IPC bus between apps and system services. (iOS is the mirror: XNU kernel → Darwin libs → Swift/Obj-C framework → your app; apps communicate via XPC instead of Binder.)

**Each layer exists for a reason — and explains a behaviour you will meet:**

| Layer | Why it exists (and what it explains) |
|-------|--------------------------------------|
| **Linux kernel + Android patches** | The **Binder** driver, **wakelocks**, ashmem, and the **Low-Memory Killer**. This is literally where "the OS may kill my process" is *implemented*. |
| **`bionic` (not glibc)** | Android's own C library: small, fast to start, **Apache-licensed** (keeps GPL out of vendor drivers). It lacks some POSIX APIs — so "it is Linux, but not *your* Linux". |
| **HAL + Project Treble** | The boundary that lets a SoC vendor's drivers update **independently** of AOSP. Before Treble, a new Android version forced the vendor to re-port drivers — which is exactly why old phones stopped updating. This is the *root cause of fragmentation* from §2. |
| **Zygote** | A pre-initialised process that `fork()`s every app, so app start-up does not pay the cost of loading the whole framework from scratch. |
| **Binder** | The single IPC channel; `ActivityManager`, `PackageManager`, and every system service are reached over it. Binder is **not in upstream Linux** — it is Android's own kernel module. |

> **Why §7 matters:** this one stack explains three separate symptoms at once — a killed process (the low-memory killer), permission enforcement (kernel UID), and fragmentation (the HAL/vendor boundary). Without the stack, debugging becomes superstition.

### 8. Developer accounts & store review
Publishing is a **pipeline with a gate**:
1. **Sign** the app (a keystore proves "this update is from the same author"). If you lose the key you cannot update the app — back it up.
2. **Upload** an AAB (Android) / build (iOS) to Play Console / App Store Connect.
3. **Automated review** (malware scan, policy checks) then **human review**.
4. **Rollout** (staged %, internal testing tracks, closed/open beta).

**AAB vs APK — and why the upload format matters.** An **APK** is the *installable* package; an **AAB (Android App Bundle)** is the *upload* format for Play. The AAB carries all code and resources, and Play generates **per-device APKs** (splits) containing only the density/language/ABI that device needs. This directly attacks the fragmentation problem from §2 — a 150 MB universal APK becomes a ~40 MB split for a given device, so the user downloads less and the developer keeps one codebase. Sideloading still uses APKs, because no store is there to do the splitting.

**Play App Signing — a partial safety net.** You upload with an *upload key*; Google re-signs with the *app signing key* it holds. That makes a lost upload key recoverable (it can be reset), instead of the old rule that losing the single signing key meant you could never update the app again. It is **not** a reason to skip backing the key up.

**Why apps get rejected (the useful list)** — grouped by type, because most first rejections are *not* code bugs:

| Type | Example | Fix |
|------|---------|-----|
| **Policy** | Undeclared data collection, over-broad permissions | The Data Safety form must match what the code actually does |
| **Technical** | Crash on launch, ANR | Test on a low-RAM device profile |
| **Metadata** | Misleading screenshots/description, using another app's trademark | Read the guidelines before submitting |
| **Legal** | Payment outside IAP for digital goods (iOS) | Use IAP |


- Missing/privacy-policy mismatch, over-broad permissions, broken functionality ("crashes on launch"), misleading metadata/screenshots, and using another app's trademark.
- Apple's **App Review Guidelines** additionally ban things like web-only "template" apps, background location without justification, and payment outside IAP for digital goods.
- **Data safety / privacy forms** are now mandatory and legally binding — they must match what your code actually does.

### 9. Shared collaboration tools (shared team tooling)
- **Git + GitHub/GitLab:** branching model (`main` protected, feature branches, PR + review), issues/boards, CI (build+test APK on every PR). See [`../../git_github/`](../../git_github/README.md) for the mechanics and [`../../git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md) for CI.
- **Communication/coordination:** chat (Slack/Teams), docs (a living spec), a board (Jira/Trello/GitHub Projects), and **version app config** — secrets (API keys) go in local files/gradle properties, never committed (Section B4).
- **Code review for mobile specifically:** watch for lifecycle leaks, main-thread I/O, missing runtime-permission handling, and hardcoded strings (i18n) — the four things reviewers catch most.

**A mobile PR review checklist** (the four leaks plus three more; the first three are *mobile-specific* — a web reviewer does not instinctively look for them):

| Check | What goes wrong if missed |
|-------|---------------------------|
| **Lifecycle** | A callback/observer outlives the Activity → leak or crash |
| **Main-thread I/O** | Disk/network on the UI thread → **ANR** (note 05) |
| **Runtime permissions** | Code assumes a grant the user denied → crash |
| **Hardcoded strings** | Cannot be localised; breaks i18n |
| **Secrets in source** | API keys in git history (Section B4) |
| **Unbounded background work** | Battery drain; Doze kills it (note 06) |
| **No empty/loading/error states** | UI breaks on the first slow network |

### 10. Android environment, Kotlin, emulator, build/run/debug
This is the **Template G** half of the note — the hands-on pipeline.

**Install:** [Android Studio](https://developer.android.com/studio) (bundles the SDK, `adb`, and the emulator). Language of choice: **Kotlin**.

**Mental model of the build pipeline:**
```
Kotlin/Java source + resources (res/)
        │  Gradle (Kotlin/Groovy DSL) — dependencies, variants
        ▼
  Compile → D8 (dex) → R8 (shrink/obfuscate)  [release only]
        ▼
  Package → APK (debug) / AAB (release)  →  zipalign + sign
        ▼
  Install on device/emulator via adb  →  ART runs it
```

**Gradle tasks you will type constantly** (the build system's public verbs):

| Task | What it does | When to use |
|------|--------------|-------------|
| `./gradlew tasks` | List available tasks | When you don't know the task name |
| `./gradlew assembleDebug` | Build the debug APK | CI artifacts, manual install |
| `./gradlew installDebug` | Build + install on the connected device | Fast inner loop |
| `./gradlew test` | Run local unit tests | Before every commit |
| `./gradlew connectedAndroidTest` | Run instrumented tests on a device | Before release |
| `./gradlew bundleRelease` | Produce the AAB for Play | Upload to Play Console |
| `./gradlew clean` | Delete build outputs | When the build is inexplicably stale |

> **Why Gradle and not `make`:** Gradle resolves a dependency graph (Maven/Google artifacts), supports **build variants** (debug/release/flavors) from one source tree, and is the same system CI drives — so "works on my machine" and "works in CI" are meant to be the *same* pipeline. (In practice they diverge: JDK version, `local.properties`, cached dependencies, and non-deterministic tests are the usual culprits.) This is the mobile-facing sibling of the YAML/CI track.

**Command cheat sheet (adb — the tool you'll live in):**
| Command | What it does | When to use |
|---------|--------------|-------------|
| `adb devices` | List connected devices/emulators | Before anything goes wrong |
| `adb install -r app-debug.apk` | Install/replace an APK | Manual install from CI artifacts |
| `adb shell pm list packages \| grep myapp` | Find your package | Confirm the app is installed |
| `adb logcat -v brief *:S MyApp:V` | Filter logs to your tag | Debugging a specific crash path |
| `adb shell dumpsys activity activities` | Dump the Activity stack | "Which Activity is on top?" |
| `adb shell am start -n pkg/.MainActivity` | Launch a component directly | Testing deep links / specific screens |
| `adb shell run-as <pkg> ls databases` | Inspect the app's private DB (debuggable) | Verify Room wrote what you think |
| `adb shell input text "hello"` | Inject text | Automating a quick manual test |
| `adb shell screencap -p /sdcard/s.png` | Screenshot | Bug reports |
| `adb pull /sdcard/s.png` | Pull a file off device | Collect artifacts |

**Practical Lab — first app, verified**
```bash
# 1. Create project in Android Studio: "Empty Views Activity", language = Kotlin
# 2. Build the debug APK from the CLI (Gradle wrapper is committed to the repo)
./gradlew assembleDebug
# 3. Start an emulator (or run from the IDE)
emulator -list-avds          # list virtual devices
emulator -avd Pixel_7_API_35 # boot one
# 4. Install and launch
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.example.myapp/.MainActivity
# 5. Watch logs for YOUR tag while you tap the screen
adb logcat -v brief *:S MyApp:V
# Expected: app window opens; logcat shows your "onCreate" log line
```

**Debugging model (three tools, three questions):**
- **Logcat** — "what happened?" (runtime timeline; filter by tag/process, `V`→`E` levels).
- **Debugger (breakpoints)** — "what is the state *right now*?" Attach, inspect variables, step.
- **Layout Inspector** — "why does it look like that?" Inspects the live view hierarchy and view attributes.
- **Profiler** — "why is it slow/janky/leaky?" CPU, memory, network timelines.

## ⚖️ Trade-offs Summary
| Decision | Choose A when... | Choose B when... |
|----------|------------------|------------------|
| Native vs cross-platform | Full platform access, native feel, long life | Reuse existing skills, fast MVP, mostly-brand UI |
| Debug technique | Reproduce locally with breakpoints | Ship logs + Play Console crash reports for field bugs |
| Permission scope | You need the feature now | You want fewer rejections → request narrowly, on demand |
| Min SDK high vs low | You want modern APIs, less testing | You want market reach, accept back-compat code |

## 🏭 Industry Corners & Connected Tech
- **Android ↔ Linux:** Android's kernel is Linux; the sandbox is a per-app UID + SELinux. `../linux/` is not optional background.
- **The JVM heritage:** Kotlin targets JVM bytecode semantics (GC, `HashMap` behavior, boxing) before ART's DEX. See [`../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md).
- **App stores ↔ economics:** the fee and gatekeeping are textbook two-sided-market/network-effect behavior — [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md).
- **Regulation:** EU DMA (2024) forced iOS sideloading + alternative payment — proof that "the store is the only channel" is a *policy* fact, not a law of physics.
- **BaaS shift:** Firebase/Amplify/Supabase moved mobile backends from "write a server" to "call a managed service" — a major cost/velocity trade-off (note 06).

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Losing the signing keystore | Can never update the app | Back it up offline; Play App Signing as a fallback |
| Requesting storage/`WRITE_EXTERNAL_STORAGE` on modern Android | Rejected or silently no-op | Use scoped storage / MediaStore (note 04) |
| Committing API keys | Leak in git history | `local.properties` + `.gitignore`, rotate the key |
| Assuming desktop runtime rules | App killed in background, alarms don't fire | Read Doze/App Standby + WorkManager docs (note 06) |
| Testing only on your emulator | Crashes on real low-end devices | Test on a low-RAM device/emulator profile; hit fragmentation |

## 🧪 Practice Tasks
- [ ] **Task 1** — Pick three apps you use daily; classify each as native / Flutter / RN / MAUI / web with justification from the table in §6. No peeking at press releases; argue from observed behavior (UI feel, update cadence, platform feature use).
- [ ] **Task 2** — Set up Android Studio, create an Empty Views Activity app, then build + install + launch it **entirely from the CLI** using only the commands in §10. Screenshot your `adb logcat` filtered to your tag.
- [ ] **Task 3** (harder) — Add a runtime permission request (e.g., `CAMERA`) to Task 2's app, deny it, and explain via Logcat what the OS did when the app tried to use the camera anyway.

## 📝 Research Journey
- **Why:** I'm building a "second brain" for engineering fundamentals; mobile is the one stack that shows how *an OS polices third-party code* — a lesson that transfers to sandboxing, least privilege, and cloud IAM.
- **Struggle:** The hardest conceptual jump was realizing the OS can kill my process at any moment and that "saving state" is not optional but a contract (onSaveInstanceState / ViewModel), which I first met here and again in note 02.
- **Aha moment:** Seeing `adb shell am start` launch a *screen by name* made the "an app is a collection of loosely-coupled components the OS wires together" model click — the same mental model as a web server routing a URL to a controller.
- **Career link:** as an AI Engineer targeting mobile-adjacent products, the constraint-mindset (battery/offline/budget) is exactly the mindset needed to scope on-device or edge ML models.

## 🔗 Related Topics
- [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md) — the concrete Android components this overview introduces
- [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md) — the kernel/permission foundations
- [`../../software_architecture/README.md`](../../software_architecture/README.md) — client/server, offline-first architecture
- [`../../economics_business/04-network-effects.md`](../../economics_business/04-network-effects.md) — why app stores concentrate so hard
- [`../../git_github/README.md`](../../git_github/README.md) — the collaboration prerequisite
- [SUGGESTED] `bridge-android-to-compose.md` — XML views vs Compose, since §6's framework table predates Compose's dominance

## 🤔 Open Questions
- [ ] Where exactly is the native/cross-platform line for a *team*, not an app? (skill leverage vs tech ceiling)
- [ ] Given EU DMA and alternative stores, does "master the Play Store" still mean what it did in 2020?
- [ ] How much of the "constraint mindset" (battery, offline) belongs to a `software_architecture/` note versus here?

## 🎓 Lecture Log

> Live teaching sessions for this chapter. The sections above hold the distilled knowledge; this log records the extra depth delivered in the lecture, the questions posed, and the argumentation that changed the note. Reverse-chronological.

### Session 5 — 2026-10-12 · §9–§10 (Team Tooling, Build & Debug) — **Chapter 1 complete**
**Depth added during the lecture:**
- §9: a **mobile PR review checklist** (7 rows) — lifecycle, main-thread I/O, runtime permissions, hardcoded strings, secrets in source, unbounded background work, missing empty/loading/error states — with the note that the first three are *mobile-specific* checks a web reviewer does not instinctively make.
- §10: a **Gradle task table** (`tasks`, `assembleDebug`, `installDebug`, `test`, `connectedAndroidTest`, `bundleRelease`, `clean`) as the build system's "public verbs", plus why Gradle over `make` (dependency graph, build variants, shared with CI) and the honest admission that local-vs-CI still diverges (JDK, `local.properties`, cache, flaky tests). Emulator-vs-physical-device framed as *fast iteration* vs *truth*.

**Chapter 1 fully covered:** §1–§10 across five sessions (2026-10-12).

**Debate prompt posed:**
- (A) §10 — if Gradle is supposedly "the same pipeline as CI", why does "works on my machine, fails in CI" still happen so often? Name the real causes.

**Student's positions:** _awaiting response_

### Session 4 — 2026-10-12 · §7–§8 (Platform Anatomy & Publishing)
**Depth added during the lecture:**
- §7 rebuilt as *"each layer exists for a reason"*: Linux kernel + Android patches (Binder driver, wakelocks, low-memory killer — where "the OS kills me" is implemented), `bionic` instead of glibc (size + Apache licence, some missing POSIX), HAL/Treble as the *root cause of fragmentation*, Zygote (fork-based fast start), Binder as the sole IPC channel not present in upstream Linux. Framed so the one stack explains process death, permission enforcement, and fragmentation together.
- §8: **AAB vs APK** (upload format vs install format; Play splits per device → answers §2 fragmentation), and **Play App Signing** as a *partial* safety net for a lost key. Rejection reasons regrouped into a Policy/Technical/Metadata/Legal table, with the lesson that most first rejections are not code bugs.

**Student's positions:** _awaiting response_

### Session 3 — 2026-10-12 · §5–§6 (Security & Framework Choice)
**Depth added during the lecture:**
- §5: the permission model reframed as a **timeline** (install-time ≤5.x → runtime 6.0 → scoped storage 10 → one-time/while-in-use 11 → granular media 13), with the observation that every step tightens a *silent access* — and the least-privilege principle it shares with Linux permissions and DB RLS.
- §5: the three distinct protection layers separated — *sandbox/scoped storage* (isolation) vs *permission* (capability) vs *encryption/Keystore* (data at rest).
- §6: the two hidden variables — the **escape hatch** (RN native modules / Flutter platform channels / MAUI handlers) as the real cost driver, and **TCO flipping over time** (cross-platform wins months 1–3, native tends to win year 3 through framework churn).

**Debate prompts posed:**
- (A) §6 — is "native feel" a legitimate requirement or a prejudice, given Flutter renders pixel-identical UI on both platforms?

**Student's positions:** _awaiting response_

### Session 2 — 2026-10-12 · §3–§4 (Platforms & Distribution)
**Depth added during the lecture:**
- Android-vs-iOS *runtime* divergence explained: Kotlin → JVM bytecode → `D8` → **DEX** → **ART** (AOT + JIT) versus Swift → LLVM → **native arm64**. Why Android chose bytecode (portability across device models/CPU architectures, free GC/reflection/dynamic loading) and iOS chose native (one controlled hardware path, deterministic performance, no GC pauses).
- The transferable rule restated: **concepts** transfer (sandbox, permission prompts, background kill-rules, declarative UI — Compose ↔ SwiftUI); **mechanisms** are per-platform tax (Binder vs XPC, DEX vs Mach-O, ART vs none).
- Store power framed as **policy, not physics** (Google \$25 one-time + sideload allowed vs Apple \$99/year + no sideload pre-DMA) — the reason regulation can move it.
- API-key hygiene: Play Console / Google Cloud / Firebase / the emulator's Google account are the leak surfaces; keys belong in `local.properties`, never committed.

**Debate prompts posed:**
- (A) Is there a way to get both portability *and* native performance? (Kotlin/Native, GraalVM, ART AOT…)
- (B) If store power is "policy, not physics", why is iOS sideloading still hard in 2026 despite the DMA? (the gate as *physics-ised policy*)

**Student's positions:** _awaiting response_

### Session 1 — 2026-10-12 · §1–§2 (Foundations)
**Depth added during the lecture** (now folded into 🧠 Mental Model and §2 above):
- The desktop-assumption vs mobile-reality table — the "the machine owns *you*" shift named explicitly, with its consequence for reading OS behaviour.
- Radio **fixed-cost** reasoning for the battery constraint (wake-up + idle tail, not payload), and why batching follows from it.
- *Sustained* power vs *peak* power as the real thermal constraint, and its consequence for algorithm choice (joules per task, not cycles per task).
- Offline-first reframed as a **data-ownership** decision: local DB = source of truth, network = replication channel.
- 48dp = ≈9mm = a fingertip → the link to **Fitts's Law** and `psychology_society/`.
- App-store gatekeeping framed as **trust infrastructure** (demand-side value) rather than only as a fee.

**Debate prompts posed to the student:**
- (A) Which of the four constraints is *hardest* for a developer moving web → mobile, and why?
- (B) Attack the claim *"gatekeeping is trust infrastructure"* — find a case where the gate demonstrably harms developers or users.

**Student's positions:** _awaiting response_

---

*An OS that can kill your app at any time is not a limitation — it is the specification. Write to it.*