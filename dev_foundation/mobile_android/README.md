# Mobile Android — Native Android Development Track

**Purpose:** The mobile layer of the university course *Mobile Application Development* (Vietnamese: "Lập trình thiết bị di động"). This folder is the whole track in one place: the operating-system/platform landscape, the fundamental Android building blocks, user-interface programming, on-device data storage, multithreading, and the specialized platform services that turn a toy app into a shipped product.

**Template:** **Template A (Tech Note)** as the spine — every chapter is fundamentally about *how a system works* (activity lifecycle, Binder IPC, the ART runtime, the storage stack). **Template G (Dev Tool & Command Note)** sections are nested inside where the chapter is about *using a tool by hand* (Android Studio, Gradle, `adb`, Room, Retrofit). Section C of `AGENTS.md` records this as `A + G`.

> **Track:** this is the **mobile track**. The language here is **Kotlin** on the Android platform (JVM + ART), with Java as the legacy/interop sibling. iOS and the cross-platform frameworks (Flutter, Xamarin, MAUI, React Native) are covered at **survey depth** inside Chapter 1 — enough to reason about the trade-offs, not enough to build a second app in each. That scope choice is deliberate: five frameworks learned at 20% is worse than one learned at 90% (see `AGENTS.md` Section N, Language Track Policy).

## Purpose in the course

| Chapter of the syllabus | Covered by |
|-------------------------|------------|
| 1. Mobile OS, platforms, dev tools (device characteristics, Android/iOS, cross-platform, dev accounts, shared tools) | `01` |
| 2. Fundamental Android components (architecture, lifecycle, MVC, Figma, core components, basic widgets) | `02` |
| 3. User-interface programming (views, navigation, lists, fragments, menus, WebView, dialogs, auto-layout) | `03` |
| 4. Local data storage & access (SharedPreferences, file system, SQLite, XML/JSON/Realm) | `04` |
| 5. Multithreading (concurrency, threads, async, Mutex, AsyncTask, REST APIs) | `05` |
| 6. Specialized services (Android Services, notifications/alarms, Firebase, FCM, Maps/GPS) + capstone | `06` |

---

## 📋 Note Index

| # | Note | Template | Core question |
|---|------|----------|---------------|
| 01 | Mobile OS, Platforms & Tools Overview | A + G | What constrains mobile software, and what does the OS/ecosystem/toolchain actually look like? |
| 02 | Android App Fundamental Components | A + G | How is an Android app put together, and when does each piece run? |
| 03 | User Interface Programming | A + G | How do I build screens that navigate, scroll, and stay responsive across devices? |
| 04 | Local Data Storage & Access | A + G | Where does state live on a device, and how do I read/write it safely? |
| 05 | Multithreading | A | How does Android keep the UI smooth while work happens elsewhere? |
| 06 | Specialized Services & Capstone | A + G | How does an app talk to the OS, the network, and the cloud — and what does the capstone assemble? |

## 📋 Note Backlog

| Priority | Planned note | Why |
|----------|--------------|-----|
| High | Jetpack Compose — the declarative UI replacement for XML | The whole note 03 stack becomes legacy if Compose is the industry default; needs its own note |
| High | Architecture patterns: MVVM, MVI, Clean Architecture | The course teaches MVC; the industry interview expects MVVM + ViewModel + LiveData/Flow |
| Medium | Jetpack ViewModel & lifecycle-aware components | The correct answer to the lifecycle problems raised in note 02 |
| Medium | Testing on Android (JUnit, Espresso, Robolectric) | Turns a course project into something maintainable |
| Medium | Publishing pipeline (signing, Play Console, Fastlane) | The missing half of note 01's store-review section |
| Low | On-device ML (ML Kit, TFLite) | Bridge to `ai_ml/` — where this track meets the career goal |
| Low | Gradle version catalogs & build variants (flavors) | Real projects need `debug`/`release`/`staging` builds |

---

## 🔗 Tech Cross-References (Section K)

- **Android ↔ Linux** — Android *is* a Linux kernel fork (with Binder, wakelocks, and no glibc). Permission enforcement, process isolation, and `/proc` are the same OS primitives. See [`../linux/01-file-system-paths-and-permissions.md`](../linux/01-file-system-paths-and-permissions.md).
- **Kotlin ↔ Java** — Kotlin runs on the JVM bytecode model; `HashMap` iteration order, boxing, and GC behavior behave like Java. See [`../dsa/02-java-for-dsa.md`](../dsa/02-java-for-dsa.md).
- **Gradle ↔ YAML/CI** — Android's build system is Groovy/Kotlin DSL, and the same CI files that run tests also build the APK. See [`../yaml_config/README.md`](../yaml_config/README.md) and [`../../git_github/11-infrastructure-as-code-and-devops-automation.md`](../../git_github/11-infrastructure-as-code-and-devops-automation.md).
- **SQLite/Room ↔ Databases** — on-device relational storage is the same schema/index/transaction theory as a server database, at a smaller scale. See [`../databases/01-relational-model-and-sql-basics.md`](../databases/01-relational-model-and-sql-basics.md).
- **REST/Firebase ↔ Architecture** — a mobile app is a client in a distributed system; offline-first, caching, and backend-as-a-service are architecture decisions. See [`../../software_architecture/`](../../software_architecture/README.md) and [`../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md`](../../ai_ml/05-mlops-lifecycle-and-deployment-architecture.md).
- **Material Design ↔ Psychology** — interaction design exploits (and must respect) human perception and habits. See [`../../psychology_society/README.md`](../../psychology_society/README.md).

## 🔗 Related Topics
- [`../README.md`](../README.md) — the umbrella engineering-fundamentals path this track plugs into
- [`../../git_github/README.md`](../../git_github/README.md) — the version-control prerequisite for every project here
- [`../databases/README.md`](../databases/README.md) — the server-side twin of note 04's storage stack
- [`../../software_architecture/README.md`](../../software_architecture/README.md) — client/server and offline-first reasoning behind notes 05–06
- [SUGGESTED] `bridge-android-to-compose.md` — bridge note: the XML view system the course teaches vs the Compose system the industry ships

---
*Created: 2026-10-11 — Planned templates: A (mechanism) + G (tooling/hands-on)*
