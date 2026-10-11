# 04 — Local Data Storage & Access

**Created Date:** 2026-10-11
**Last Updated:** 2026-10-11
**Tags:** `#Android`, `#Persistence`, `#SharedPreferences`, `#SQLite`, `#Room`, `#ScopedStorage`, `#JSON`, `#Realm`
**References:** Android Developers — "Data and file storage overview", "Save data using SQLite", "Scoped storage", "Room persistence library"; SQLite docs; Realm docs; `../databases/01-relational-model-and-sql-basics.md`
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** [`02-android-app-fundamental-components.md`](02-android-app-fundamental-components.md), [`../databases/01-relational-model-and-sql-basics.md`](../databases/01-relational-model-and-sql-basics.md)

---

## 📌 Key Takeaways (TL;DR)
- On mobile, **the local store is often the source of truth**, not a cache. Offline-first means the app reads/writes locally and *syncs* to the network (which may be absent) — the opposite instinct from a web page.
- **Pick the lightest tool that fits**: key/value → `SharedPreferences`/DataStore; blobs/files → the file system; structured+queryable → SQLite (via **Room**).
- **Scoped storage (Android 10+) changed the rules**: apps cannot read the whole filesystem. You own your private dir; shared files go through MediaStore/SAF.
- **Room is not "SQLite with sugar"** — it is a compile-time-checked ORM: annotations generate SQL, verify it against the schema, and return typed objects + `Flow` for reactivity.
- **Format ≠ database.** JSON is a serialization format (small config/API payloads); XML is verbose/legacy; Realm is an object DB. Choose by access pattern, not fashion.

## 🧠 Mental Model
Lay out the storage options on two axes — **structure** (how queryable?) and **cost** (how heavy?):

```
 heavy, queryable ┌───────────────────────────────┐
                  │  SQLite (Room)  ·  Realm      │
                  │  (relational / object DB)     │
                  ├───────────────────────────────┤
                  │  Files (images, downloads)    │
                  ├───────────────────────────────┤
                  │  SharedPreferences / DataStore │
 light, simple    └───────────────────────────────┘
                     key/value          structured
```

> **The rule of thumb:** if you need to *query across records by more than one key*, you want a database. If you just remember "the user's theme" or "last sync time", a key/value store is correct and cheaper. Using SQLite for one boolean is the mobile equivalent of standing up a Postgres server to store a config flag.

## 🔬 Detailed Notes

### 1. Why local data (and the offline-first mindset)
On the web, the server usually owns state; the browser caches. On mobile:
- **Network is intermittent** — tunnels, planes, rural areas. If the app can't work offline, it's broken half the time.
- **Latency & battery** — a local read is microseconds and free; a network round trip is 100s of ms and drains the radio.
- **User expectation** — apps open instantly with the last-known data, then refresh.

**Offline-first architecture (the pattern notes 05–06 build on):**
```
User action ──► LOCAL DB (write, instant UI) ──► queue sync ──► server (eventually)
                      ▲                                              │
                      └──────────── background pull / push ◄─────────┘
```
The local DB is the single source of truth for the UI; the network is a *synchronization channel*. Conflicts (two edits) are the hard problem — resolved by timestamps, version numbers, or server authority.

### 2. Storage options overview
| Option | Data shape | Best for | Size/limits | Multi-thread safe |
|--------|-----------|----------|-------------|-------------------|
| `SharedPreferences` / DataStore | key → primitive/string | Settings, flags, small state | small | DataStore: yes (Flow) |
| Internal files | bytes | app-private files, caches | device storage | manual |
| External/shared (MediaStore, SAF) | bytes | user-visible files, media | device storage | system-mediated |
| SQLite / Room | rows + columns | structured, queryable, relational | large | SQLite yes; Room enforces off-main-thread |
| Realm | objects | fast object graphs, reactive | large | yes |

### 3. `SharedPreferences` (and the modern DataStore)
```kotlin
val prefs = getSharedPreferences("settings", Context.MODE_PRIVATE)

// write (commit() = synchronous, apply() = async, preferred)
prefs.edit().putBoolean("dark_mode", true).apply()

// read
val darkMode = prefs.getBoolean("dark_mode", false)
```
**Mental model:** it's an in-memory `Map` backed by an XML file, loaded once per process. Fine for a handful of flags; **not** for lists or large data.
**Trap:** it's stored in plaintext inside your app's sandbox — never put secrets/tokens here unencrypted (use `EncryptedSharedPreferences` / Keystore).
**Modern replacement — Jetpack DataStore:** type-safe, `Flow`-based, no blocking. Prefer it for new code.

### 4. File system & scoped storage (Android 10+)
Three storage zones:
```
/data/data/<pkg>/files/        ← internal, PRIVATE, removed on uninstall   (Context.filesDir)
/data/data/<pkg>/cache/        ← internal cache, OS may evict              (cacheDir)
/storage/emulated/0/...        ← SHARED (Photos, Downloads, DCIM)         ← via MediaStore/SAF only
```
**Scoped storage** means: your app can always read/write its **own** internal dir without permission; to touch *shared* files it must go through the **MediaStore** (for media) or the **Storage Access Framework** (SAF, for documents). Broad `WRITE_EXTERNAL_STORAGE` is gone on modern Android.

```kotlin
// Internal private file — no permission needed
val file = File(filesDir, "notes.txt")
file.writeText("hello")                          // small; for big data use streams
val text = file.readText()

// User-picked document — the SAF way (no permission, user grants access)
val launcher = registerForActivityResult(
    ActivityResultContracts.CreateDocument("text/plain")
) { uri -> uri?.let { contentResolver.openOutputStream(it)!!.use { os -> os.write("data".toByteArray()) } } }
launcher.launch("my-notes.txt")
```
**Rules that prevent 80% of storage bugs:** use `filesDir`/`cacheDir` for app data; always use `try/finally` (`use {}`) to close streams; never hardcode `/sdcard`; handle `IOException`; put large media in cache with an eviction policy.

### 5. SQLite via Room (the structured store)
**Why Room over raw `SQLiteOpenHelper`:** compile-time SQL verification, no boilerplate Cursors, mapping to objects, migration tooling, and coroutine/`Flow` integration.

```kotlin
@Entity(tableName = "notes")
data class Note(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val title: String,
    val body: String,
    val updatedAt: Long
)

@Dao
interface NoteDao {
    @Query("SELECT * FROM notes ORDER BY updatedAt DESC")
    fun observeAll(): Flow<List<Note>>                 // reactive: UI updates on change

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun upsert(note: Note): Long

    @Query("DELETE FROM notes WHERE id = :id")
    suspend fun deleteById(id: Long)
}

@Database(entities = [Note::class], version = 1)
abstract class AppDb : RoomDatabase() {
    abstract fun notes(): NoteDao
    companion object {
        @Volatile private var INSTANCE: AppDb? = null
        fun get(context: Context): AppDb = INSTANCE ?: synchronized(this) {
            INSTANCE ?: Room.databaseBuilder(context, AppDb::class.java, "app.db").build()
                .also { INSTANCE = it }
        }
    }
}
```
**Usage is off the main thread by contract:**
```kotlin
lifecycleScope.launch {
    val dao = AppDb.get(applicationContext).notes()
    dao.observeAll().collect { list -> adapter.submitList(list) }   // reactive read
    dao.upsert(Note(title = "Hi", body = "world", updatedAt = System.currentTimeMillis()))
}
```
**Concept bridges to server databases:** entities ≈ tables, `@Query` ≈ SQL, migrations ≈ `ALTER TABLE`, `@Relation`/`@Transaction` ≈ joins. The **same indexing/schema reasoning** applies at smaller scale — see [`../databases/01-relational-model-and-sql-basics.md`](../databases/01-relational-model-and-sql-basics.md) and [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md).

**Room pitfalls:** queries run on the **main thread throw** (by design); `@Query` SQL is checked at compile time (great, until you rename a column and forget the query); migrations must be written, or `fallbackToDestructiveMigration` wipes user data.

### 6. Data formats: XML vs JSON vs Realm
| Format | Shape | Strengths | Weaknesses | Use it for |
|--------|-------|-----------|-----------|-----------|
| **XML** | tree, verbose | Schema (XSD), Android resources, legacy | Verbose, slower parse | Android `res/`, legacy APIs/config |
| **JSON** | tree, compact | Ubiquitous, readable, every API | No schema, no types, larger than binary | API payloads, config, small structured data |
| **Realm** | object DB | Fast, reactive, object-graph native | Not SQL, extra dependency, license/size | Local object-heavy apps wanting reactivity |

**JSON parsing on Android — two worlds:**
```kotlin
// 1) kotlinx.serialization (Kotlin-native, compile-time, no reflection)
@Serializable data class User(val id: Long, val name: String, val email: String?)
val user = Json.decodeFromString<User>(jsonString)
val back = Json.encodeToString(user)

// 2) Gson / Moshi (reflection-based, Java-era)
val u: User = Gson().fromJson(jsonString, User::class.java)
```
**Why this matters:** JSON is the *lingua franca* of REST APIs (note 05) and Firebase (note 06), so it's not optional — but "parsing JSON" is where you meet null-safety (`String?`), unknown fields, and schema drift. `kotlinx.serialization` catches shape mismatches at compile time; Gson fails at runtime with `JsonSyntaxException`.

> **Format vs database, one more time:** you can store JSON *inside* a SQLite text column (common for flexible payloads), but that's not a reason to use JSON *as* your database — you lose query/index capabilities.

## ⚖️ Trade-offs Summary
| Decision | Option A | Option B |
|----------|----------|----------|
| Small settings | SharedPreferences (simple) | DataStore (typed, async, recommended) |
| Structured data | Room (typed, reactive, safe) | Raw SQLite (max control, more boilerplate) |
| Object graph | Room + relations | Realm (native object graph) |
| API data | Parse on the fly | Cache JSON in Room for offline-first |
| File access | Internal `filesDir` (private) | SAF/MediaStore (shared, user-mediated) |

## 🏭 Industry Corners & Connected Tech
- **Room ↔ server DB theory:** identical relational concepts (schema, index, transaction, migration). Learn it here, reuse it server-side and vice-versa. [`../databases/`](../databases/README.md).
- **Offline-first ↔ distributed systems:** conflict resolution, eventual consistency, and idempotent sync are distributed-systems problems in your pocket. [`../../software_architecture/`](../../software_architecture/README.md).
- **Encryption ↔ security:** at-rest data and Keystore are the mobile face of the web's session/token security. [`../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md).
- **Battery/IO:** every disk write is a battery cost; batching writes (and using `apply()`/transactions) is a mobile-specific optimization.
- **Sync ↔ ML data pipelines:** the local store is a data source/feature store on-device — a bridge to `ai_ml/`.

## ⚠️ Common Pitfalls
| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Room query on main thread | `IllegalStateException` | `suspend`/`Flow` + coroutines (note 05) |
| Broad storage permission | Rejected/no-op on Android 10+ | Scoped storage: MediaStore/SAF |
| Forgetting migrations | Data wiped or crash on upgrade | Write migrations; test them |
| Storing tokens in `SharedPreferences` | Credential leak | `EncryptedSharedPreferences`/Keystore |
| Unclosed streams | `IOException`, file corruption | `use {}` everywhere |
| Assuming a fresh DB every launch | "Data disappeared" | Check install/uninstall + migration |
| `notifyDataSetChanged` on a Room `Flow` | Full-list rebind | Use `ListAdapter`/`DiffUtil` (note 03) |

## 🧪 Practice Tasks
- [ ] **Task 1** — Build a notes app: Room entity + DAO + database, a `RecyclerView` observing a `Flow`, and upsert/delete. Verify the list updates live and survives an app restart. Inspect the DB with `adb shell run-as <pkg> ls databases` and pull it.
- [ ] **Task 2** — Add a theme toggle stored in `SharedPreferences` (or DataStore). Prove it persists across restarts, then migrate it to `EncryptedSharedPreferences` and explain what changed on disk.
- [ ] **Task 3** (harder) — Add a schema migration `v1 → v2` (new column). Prove old data survives. Then deliberately break it and observe the failure to understand why migration discipline matters.

## 📝 Research Journey
- **Why:** Persistence is where an app stops being a toy — and mobile persistence is a compressed version of the database theory I study separately, so it's a perfect cross-domain bridge.
- **Struggle:** Scoped storage was the confusing part; the mental shift from "the filesystem is mine" to "I have a private dir and must ask for shared files" took re-reading the platform docs.
- **Aha moment:** Mapping Room's `@Entity`/`@Dao`/`@Query` onto the SQL and index concepts from `databases/` made both topics click simultaneously — same theory, smaller scale.
- **Career link:** offline-first + sync + conflict resolution is exactly the data-engineering reasoning behind ML feature stores and edge inference, which is where my AI interests sit.

## 🔗 Related Topics
- [`../databases/01-relational-model-and-sql-basics.md`](../databases/01-relational-model-and-sql-basics.md) — the relational theory Room implements
- [`../databases/08-db-bottlenecks-indexing-and-caching.md`](../databases/08-db-bottlenecks-indexing-and-caching.md) — indexing/caching at scale
- [`03-user-interface-programming.md`](03-user-interface-programming.md) — `RecyclerView` observing Room flows
- [`05-multithreading.md`](05-multithreading.md) — why Room forces you off the main thread
- [`../../software_architecture/README.md`](../../software_architecture/README.md) — offline-first & sync as architecture
- [SUGGESTED] `bridge-offline-sync-conflict-resolution.md` — the sync/conflict problem deserves its own note

## 🤔 Open Questions
- [ ] When does an app outgrow Room and need a real sync engine (e.g., a CRDT / offline-sync framework)?
- [ ] How should we choose between caching API responses as JSON vs normalizing them into Room tables?
- [ ] What's the minimal correct sync protocol for a two-device conflict, and how much of it can we reuse from server-side databases?

---
*On mobile the database is the source of truth and the network is the mail carrier — not the other way around.*
