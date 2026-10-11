# 06 — Specialized Services & Capstone

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-11
**Tags:** `#Android`, `#Services`, `#WorkManager`, `#Notifications`, `#Firebase`, `#FCM`, `#Maps`, `#GPS`, `#Capstone`
**References:** Android Developers — "Services overview", "Notifications", "WorkManager", "AlarmManager", "Fused Location Provider"; Firebase docs (Auth, Realtime Database, Firestore, Storage, Analytics, Cloud Messaging); Google Maps Platform docs
**Status:** Draft
**Difficulty:** Intermediate → Advanced
**Prerequisites:** [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md), [`05-multithreading.md`](05-multithreading.md)

---

## 📌 Key Takeaways (TL;DR)
- **A Service is not "a background thread".** It runs on the main thread by default; it's a *component the OS keeps alive*, and since Android 8 the OS aggressively restricts background work — you must justify and schedule it.
- **Choose the right background tool by the *kind* of work**: foreground service (user-visible now), `WorkManager` (deferrable, guaranteed), `AlarmManager` (exact time), coroutine in a lifecycle (in-app). Using the wrong one is the #1 cause of "my background task doesn't run".
- **Notifications are the OS-owned channel to the user**, and their power (heads-up, badges, taps) is gated by **channels + permission** (Android 13+ `POST_NOTIFICATIONS`).
- **Firebase is Backend-as-a-Service**: it replaces a hand-written server for auth, realtime data, files, and analytics — trading control/portability for speed.
- **FCM is push, not pull**: the server sends a message, Google's infrastructure wakes your app, and *you* decide what to do — you don't poll.

## 🧠 Mental Model
Think of "background" as a **ladder of permission** — the OS gives you more freedom the more the user can see and expect it:

```
 More freedom  ▲  Foreground service + ongoing notification   ("I'm active, user knows")
   |           │  Bound service                               (client connected to me)
   |           │  WorkManager (deferrable, OS-scheduled)      ("do this soon, whatever")
   |           │  AlarmManager (exact time)                   ("wake at 7:00 sharp")
 Less freedom  ▼  Plain background thread / coroutine        (only while app alive)
```

> **The rule that explains 90% of "background doesn't work" questions:** since Android 8 there is no such thing as a free-running background service. Every long-lived background job is a *contract* with the OS: it must be user-visible (foreground), client-bound, or deferred to the OS scheduler.

## 🔬 Detailed Notes

### 1. Android Services — the four shapes
| Type | Started by | Lifetime | Use for |
|------|-----------|----------|---------|
| **Started service** | `startService()` | Until `stopSelf()`, or killed | Legacy long tasks (now use foreground/WorkManager) |
| **Foreground service** | `startForeground()` + notification | Until stopped; user-visible | Music playback, navigation, active uploads |
| **Bound service** | `bindService()` | Until last client unbinds | Client↔server in-process/IPC communication |
| **IntentService** | `startService()` | Self-stops when queue drains | **Deprecated** → replaced by `WorkManager`/coroutines |

```kotlin
class UploadService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Must promote to foreground within ~5s or the OS kills you (Android 12+ forbids
        // starting foreground services from the background in some cases).
        startForeground(NOTIF_ID, buildNotification("Uploading…"))
        return START_STICKY     // or START_NOT_STICKY / START_REDELIVER_INTENT
    }
    override fun onBind(intent: Intent?): IBinder? = null
}
```
**Return flags matter:** `START_STICKY` ("restart me if killed, `intent` may be null"), `START_NOT_STICKY` ("don't bother"), `START_REDELIVER_INTENT` ("restart and re-deliver the last intent"). Choosing wrong produces either zombie services or lost work.

**BroadcastReceiver** — reacts to system/app events. Since Android 8, most **implicit** broadcasts can't be manifest-declared (battery/performance); register at runtime or use `WorkManager`:
```kotlin
// Runtime receiver — lives as long as the registering component
val receiver = object : BroadcastReceiver() {
    override fun onReceive(ctx: Context, intent: Intent) { /* react */ }
}
ContextCompat.registerReceiver(this, receiver,
    IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION), ContextCompat.RECEIVER_NOT_EXPORTED)
```

### 2. Notifications & Alarms
**Notifications** are the visible half of background work:
```kotlin
// Android 8+: everything is in a channel, and channels carry the user's importance settings
val channel = NotificationChannel("uploads", "Uploads", NotificationManager.IMPORTANCE_DEFAULT)
getSystemService(NotificationManager::class.java).createNotificationChannel(channel)

val notif = NotificationCompat.Builder(this, "uploads")
    .setSmallIcon(R.drawable.ic_upload)
    .setContentTitle("Upload complete")
    .setContentText("3 files uploaded")
    .setContentIntent(pendingIntentToOpenApp())   // tap → open the app
    .setAutoCancel(true)
    .build()
NotificationManagerCompat.from(this).notify(NOTIF_ID, notif)
```
**Permission gate:** on Android 13+ you must request `POST_NOTIFICATIONS` at runtime — if the user denies, `notify()` silently does nothing. Design must handle the denied case (in-app banners).

**Three scheduling tools, three intents:**
| Tool | Guarantees | Use for |
|------|-----------|---------|
| `WorkManager` | Runs *eventually* (survives reboot/process death), with constraints | Sync, upload, periodic cleanup |
| `AlarmManager` (`setExactAndAllowWhileIdle`) | Fires at an exact time (may be deferred in Doze) | Alarms, calendar reminders |
| Coroutine in a scope | Runs *now*, while app alive | In-app async (note 05) |

```kotlin
// WorkManager: the right default for deferrable work
val request = PeriodicWorkRequestBuilder<SyncWorker>(15, TimeUnit.MINUTES)
    .setConstraints(Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)   // "only when online"
        .setRequiresBatteryNotLow(true)
        .build())
    .build()
WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "sync", ExistingPeriodicWorkPolicy.KEEP, request)
```
> **Doze & App Standby:** when the phone sleeps, the OS batches alarms and blocks network. `WorkManager` respects this by design; a naive `Handler.postDelayed` does not. Repeat after me: *network work belongs in WorkManager, not a timer.*

### 3. Firebase — Backend-as-a-Service
Firebase bundles the managed services a mobile backend usually needs. The trade: **speed of development for vendor lock-in and less control.**

| Firebase service | Replaces | Notes |
|------------------|----------|-------|
| **Authentication** | Your signup/login backend | Email/password, Google/Apple, phone OTP; hands you a JWT-like ID token |
| **Realtime Database** | A WebSocket server | JSON tree, millisecond sync, great for presence/chat |
| **Cloud Firestore** | A document DB server | Collection/document model, queries, offline cache, scales better than RTDB |
| **Cloud Storage** | File/object hosting | Upload/download blobs (images, videos) |
| **Analytics** | Product analytics backend | Event logging + funnels, free and deep |
| **Cloud Messaging (FCM)** | Your push server | Push notifications (see §4) |

**Firebase Auth (with coroutines):**
```kotlin
val auth = FirebaseAuth.getInstance()
auth.signInWithEmailAndPassword(email, password)
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            val user = auth.currentUser
            // user?.getIdToken(true) → bearer token for your own backend
        } else showError(task.exception)
    }
```
**Realtime Database vs Firestore — the choice:**
```
Realtime DB:  a single big JSON tree, sync everything, cheap simple reads, weaker queries
Firestore:    documents + collections, expressive queries + indexes, better scaling, offline
```
**Security rules** are Firebase's authorization layer — the equivalent of a database's row-level security. They are *not optional*: `<project>.firebaseio.com` left in test mode is a very common data-leak story.

```javascript
// Firestore rule sketch — users read/write only their own documents
match /users/{uid} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

### 4. GCM → FCM & remote notifications
**GCM (Google Cloud Messaging)** was the old push service; it is retired, replaced by **FCM (Firebase Cloud Messaging)**. The architectural idea is unchanged: your server talks to Google, Google talks to the device.
```
Your server ──HTTP──► FCM ──► Google Play Services on device ──► your app
                        (holds the long-lived socket; no polling)
```
```kotlin
// Receiving: a service extends FirebaseMessagingService
class MyMessaging : FirebaseMessagingService() {
    override fun onMessageReceived(msg: RemoteMessage) {
        msg.data["type"]?.let { if (it == "chat") showChatNotification(msg) }
    }
    override fun onNewToken(token: String) {
        // Send this token to YOUR server so it can target this device
    }
}
```
> **Push ≠ pull:** you never poll for notifications. The OS keeps a single socket; FCM wakes your app only when there's a message. This is the battery-efficient design that makes notifications viable — and the reason a "check every 5 seconds" implementation is an anti-pattern.

**Two message types:** *notification messages* (handled by the OS, shown automatically) vs *data messages* (delivered to `onMessageReceived`, you build the notification). Data messages give control; notification messages work even when the app is killed.

### 5. Maps & GPS
**Fused Location Provider** (Google Play Services) blends GPS, Wi-Fi, and cell to give the best accuracy/battery trade-off. You *request a quality*, not a sensor.
```kotlin
val priority = Priority.PRIORITY_BALANCED_POWER_ACCURACY  // vs HIGH_ACCURACY (battery!)
val request = LocationRequest.Builder(priority, 10_000L)
    .setMinUpdateIntervalMillis(5_000L).build()

val client = LocationServices.getFusedLocationProviderClient(this)
client.requestLocationUpdates(request, locationCallback, Looper.getMainLooper())
```
**Maps (Maps SDK for Android):**
```kotlin
// In XML: com.google.android.gms.maps.SupportMapFragment
override fun onMapReady(map: GoogleMap) {
    map.addMarker(MarkerOptions().position(LatLng(21.02, 105.85)).title("Marker"))
        .showInfoWindow()
    map.moveCamera(CameraUpdateFactory.newLatLngZoom(LatLng(21.02, 105.85), 14f))
}
```
**The three things everyone gets wrong:**
1. **API key ≠ free.** A Google Maps key must be restricted (by app package + SHA-1), or someone will bill your account.
2. **Location is a permission + a battery cost.** Request `ACCESS_FINE_LOCATION` at runtime; prefer fine only if accuracy demands it. Background location needs special justification and gets store scrutiny.
3. **Don't poll; subscribe.** Use `requestLocationUpdates` with a sensible interval, and remove updates in `onPause`/`onStop` (lifecycle!).

### 6. Capstone project — the synthesis
The final project assembles chapters 1–6 into one app. A sound scope:

**Suggested domains (each exercises the full stack):**
- **Offline-first note/task app** — Room (04) + WorkManager sync (06) + notifications (06) + a REST or Firebase backend (05/06).
- **Location-based delivery/messaging app** — Maps/GPS (06) + FCM push (06) + Firebase Auth/Firestore (06) + RecyclerView lists (03).
- **Media/recommendation app** — Retrofit (05) + image loading + Analytics (06), with a simple on-device ML tie-in (bridge to `ai_ml/`).

**A capstone checklist (what "complete" means):**
```
[ ] Architecture: separated UI / data (repository) layers; no God Activity
[ ] Screens: ≥ 3 screens with navigation + back stack + deep link
[ ] Data: local persistence (Room) as source of truth, offline-capable
[ ] Network: at least one real REST or Firebase integration with error handling
[ ] Background: one correct background job (WorkManager foreground/sync)
[ ] Push: FCM or scheduled notification working end-to-end
[ ] Auth: real signup/login (Firebase Auth or your backend) + logout
[ ] Permissions: requested at runtime, gracefully denied
[ ] Quality: no leaks on rotation, no main-thread I/O, no ANR
[ ] Portfolio: versioned on GitHub/GitLab, real README + screenshots + release APK
```

## ⚖️ Trade-offs Summary
| Decision | Option A | Option B |
|----------|----------|----------|
| Background work | WorkManager (deferrable, guaranteed) | Foreground service (user-visible, immediate) |
| Backend | Firebase/BaaS (fast, locked-in) | Own REST backend (control, portable) |
| Realtime data | Realtime Database (simple, sync-all) | Firestore (queryable, scalable) |
| Location accuracy | `HIGH_ACCURACY` (precise, costly) | `BALANCED_POWER` (cheap, coarse) |
| Push content | Notification message (OS-built) | Data message (you build) |

## 🏭 Industry Corners & Connected Tech
- **BaaS ↔ economics:** Firebase/Amplify/Supabase shift backend cost from "engineer time" to "usage bill" — a classic build-vs-buy decision ([`../../economics_business/`](../../economics_business/README.md)).
- **Push at scale ↔ distributed systems:** FCM/APNs are massive fan-out systems; the mobile developer just sees the edge. [`../../software_architecture/`](../../software_architecture/README.md).
- **Security rules ↔ row-level security:** the same authz thinking as Postgres RLS / server guards ([`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md)).
- **Location ↔ privacy & ethics:** background location and device identifiers are exactly what GDPR/BIPA regulate — a bridge to [`../../ai_ml/07-affective-computing-and-emotion-recognition.md`](../../ai_ml/07-affective-computing-and-emotion-recognition.md) and `psychology_society/`.
- **On-device services ↔ ML:** Fused Location, ML Kit, and TFLite show the platform providing models/services as OS features — the frontier where this track meets `ai_ml/`.

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Background service that "never runs" | Task silently skipped | Use WorkManager/foreground service, not a raw background service |
| No notification channel (Android 8+) | Notification never shows | Create the channel first |
| Forgetting `POST_NOTIFICATIONS` (13+) | `notify()` no-op | Runtime permission request |
| Unrestricted Maps API key | Unexpected billing | Restrict by package + SHA-1 |
| Firebase test-mode rules | Public data leak | Write and test security rules |
| Polling instead of push | Battery drain, stale data | Use FCM |
| Continous location updates never stopped | Battery drain, ANR reports | Remove updates in `onStop` |
| Background location without justification | Store rejection | Only if core; document the use |

## 🧪 Practice Tasks
- [ ] **Task 1** — Implement a `PeriodicWorkRequest` that syncs local Room data to a mock API every 15 minutes (constraint: network connected). Verify with `adb shell dumpsys jobscheduler` that it's scheduled, then with logs that it runs.
- [ ] **Task 2** — Send yourself a data-message push via the FCM console; build a custom notification in `onMessageReceived` and open a specific screen on tap (deep link back to note 03).
- [ ] **Task 3** (capstone) — Execute the capstone checklist end-to-end for one of the suggested domains on GitHub. This is the graded artifact.

## 📝 Research Journey
- **Why:** This is where all five prior notes stop being separate and become one product; I wanted the "glue" chapter, because that's the actual skill employers pay for.
- **Struggle:** The background-execution rules (Doze, foreground-service start restrictions, implicit-broadcast bans) felt like arbitrary gates until I read them as *battery governance* — the OS protecting the user from the developer.
- **Aha moment:** Learning that push is not polling — the OS holds one socket for *all* apps — reframed notification infrastructure from "magic" to a shared efficiency layer.
- **Career link:** BaaS/FCM/GPS are the product-facing surface where AI features ship (recommendations, notifications, location-aware models); the capstone is my first portfolio artifact at that level.

## 🔗 Related Topics
- [`05-multithreading.md`](05-multithreading.md) — the in-app async this chapter schedules around
- [`04-local-data-storage-and-access.md`](04-local-data-storage-and-access.md) — the local source-of-truth WorkManager syncs
- [`../../software_architecture/README.md`](../../software_architecture/README.md) — where Firebase/BaaS sits in system design
- [`../security/02-authorization-and-dos-defense.md`](../security/02-authorization-and-dos-defense.md) — Firebase rules ↔ authorization
- [`../../ai_ml/07-affective-computing-and-emotion-recognition.md`](../../ai_ml/07-affective-computing-and-emotion-recognition.md) — privacy/ethics of location & device data
- [SUGGESTED] `bridge-android-to-on-device-ml.md` — ML Kit/TFLite from an Android client

## 🤔 Open Questions
- [ ] Where's the crossover point where building a small REST backend beats Firebase's lock-in?
- [ ] How should an offline-first capstone resolve sync conflicts without a sync framework?
- [ ] Can FCM data messages be used as a reliable transport for a chat app, or does that always need a socket/Realtime DB?

---
*Services are the app's hands reaching outside itself; the OS lets them work only when the user would expect them to.*
