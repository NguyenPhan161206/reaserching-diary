# Forms, Fetch & Multipart Upload — Sending Data from the Browser

**Created Date:** 2026-10-10
**Last Updated:** 2026-10-10
**Tags:** `#Forms`, `#Fetch`, `#Upload`, `#Multipart`, `#FormData`, `#AJAX`, `#HTTP`
**References:** [MDN — Sending form data](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data) · [MDN — FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) · [MDN — Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) · [MDN — Constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation) · [expressjs.com — multer/error handling](https://expressjs.com/en/guide/error-handling.html)
**Status:** Draft
**Difficulty:** Intermediate
**Prerequisites:** HTML structure & forms ([`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md)); JavaScript language core ([`04-javascript-language-core.md`](04-javascript-language-core.md)); DOM & event loop ([`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md))

---

## 📌 Key Takeaways (TL;DR)
- **Nothing happens until the wire.** A form and `fetch` are two front-ends to the *same* event: the browser builds an HTTP request and sends it. There is no third magic path.
- **The `name` attribute is the key the server receives.** A control without a `name` is never submitted — the single most common "my field is empty on the server" bug.
- **Two encodings ride the wire:** `application/x-www-form-urlencoded` (text-only `k=v&k2=v2`) and `multipart/form-data` (required for files; uses a `boundary`). A file **cannot** ride urlencoded.
- **`fetch` does not reject on HTTP errors.** A `404` or `500` still resolves; only a *network* failure rejects. Always check `response.ok`.
- **With `FormData`, never set `Content-Type` yourself** — the browser must add `; boundary=…`. Hand-setting it drops the boundary and the server cannot parse the file.
- **Client-side validation is UX, never security.** Anyone can bypass it with DevTools; the server must validate again (see [`../../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)).

## 🧠 Detailed Notes

### 1. Core Concepts (Mental Model)

HTML forms are the **classic** way the browser ships user data to a server. `fetch` is the **modern JavaScript** way to do the same thing without leaving the page. Both end at the exact same checkpoint: **an HTTP request with a method, a URL, headers, and a body**. The browser does not "submit to JavaScript" or "submit to the server" — there is only one pipe.

```
  FORM (declarative)                     FETCH (imperative)
  <form action method enctype>           fetch(url, { method, headers, body })
          │                                         │
          └──────────────► HTTP REQUEST ◄───────────┘
                       method · URL · headers · body
                                   │
                                   ▼
                          SERVER (Express, see web_server/)
```

**Why this framing matters:** students treat a form and an AJAX call as unrelated topics, then cannot debug either. Once you see both as "produce an HTTP request", the questions become concrete — *what method? what `Content-Type`? what does the body look like on the wire?* — and those are answerable in DevTools' Network tab.

This note is the **browser half** of upload. The server half (Express routes, `multer`, saving files) lives in [`../../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md).

**The full round trip (mechanism).** Whether the trigger is a `<button type="submit">` or `fetch`, the browser performs the same ordered steps:

1. **Collect.** The browser walks the form's controls and builds a list of `name → value` pairs. Controls that are `disabled` or lack a `name` are skipped; a `checkbox`/`radio` contributes only if checked.
2. **Encode.** It picks an encoding from `enctype` (or, for `fetch`, from the `body` type) and serializes the pairs into the body: `k=v&k2=v2` for urlencoded, boundary-separated parts for multipart, or a JSON string if you sent one.
3. **Send.** It issues an HTTP request: method from `method`, path from `action`, a `Content-Type` header matching the encoding (plus the auto-generated `boundary` for multipart), and the encoded body.
4. **Receive.** The server parses the body according to that `Content-Type`, produces a response with a status code, and returns it.
5. **React.** A classic submit **navigates** the browser to the response (reloading the page); `fetch` **hands the response to JavaScript** so the page stays put.

Steps 1–4 are identical for both paths — only step 5 differs. That is why a field that is "missing" on the server is almost always a step-1 problem (no `name`, or `disabled`), and a file that is "unparseable" is almost always a step-2/3 problem (wrong `enctype`, or a hand-set `Content-Type` with no boundary). Debug the step, not the symptom.

---

### 2. The Form Element

```html
<form action="/subscribe" method="post" enctype="multipart/form-data">
  <label for="email">Email</label>
  <input type="email" id="email" name="email" required>

  <label for="plan">Plan</label>
  <select id="plan" name="plan">
    <option value="free">Free</option>
    <option value="pro" selected>Pro</option>
  </select>

  <label for="notes">Notes</label>
  <textarea id="notes" name="notes" rows="3"></textarea>

  <button type="submit">Subscribe</button>
</form>
```

- **`action`** — the URL the request goes to. Omitted → the current page.
- **`method`** — `GET` (data in the URL query string, idempotent, for searches) or `POST` (data in the request body, for creating/changing state). GET must never change state: it is bookmarkable, cacheable, and logged.
- **`enctype`** — how the body is encoded. Default `application/x-www-form-urlencoded`; set `multipart/form-data` when the form contains a file.
- **`input type`** — drives both the keyboard (`type="tel"` shows a phone keypad) and built-in validation (`type="email"` checks shape). Common: `text`, `email`, `number`, `date`, `password`, `checkbox`, `radio`, `file`, `hidden`.
- **`name`** — **the key the server receives**. The browser sends `name=value`; a control with no `name` is *silently skipped*. The `id` is only for the `<label for>` link and CSS/JS.
- **`<label for="email">`** — clicking the label focuses the input, and screen readers announce it. This is accessibility, not decoration.

Three control distinctions that trip people up:

- **`checkbox` vs `radio`.** A `radio` group shares one `name` and only one can be selected; each contributes its `value` when checked. A `checkbox` usually has its own `name` — an unchecked box is simply **absent** from the payload (not `false`), so your server must default it.
- **`disabled` vs `readonly`.** A `disabled` control is greyed out and **not submitted at all**; a `readonly` control is not editable but **is** submitted. If a value must reach the server but not be changed by the user, use `readonly` (or a hidden input), never `disabled`.
- **`hidden`.** `<input type="hidden" name="id" value="42">` carries data the user never sees (row IDs, CSRF tokens) with the same submission rules as any other input. It is *not* secret — anyone can read it in the page source — so it is for identifiers, not for credentials.

> **Syllabus anchor:** this is the markup side of the client/server data flow; the server route it hits is explained in [`../../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md).

---

### 3. Two Encodings on the Wire

The `enctype` decides the `Content-Type` header and the body format.

**A. `application/x-www-form-urlencoded` (the default)**

```
POST /subscribe HTTP/1.1
Host: localhost:3000
Content-Type: application/x-www-form-urlencoded

email=a%40b.com&plan=pro&notes=hello+world
```

Each field is `name=value`, pairs joined by `&`, and **special characters are percent-encoded** (`@` → `%40`, a space → `+` or `%20`). It is compact and text-only — perfect for short fields.

**B. `multipart/form-data`**

```
POST /upload HTTP/1.1
Host: localhost:3000
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="title"

Holiday photo
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="me.png"
Content-Type: image/png

<raw binary bytes of the PNG file>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

The **boundary** is a random string that separates parts; the browser generates it and declares it in `Content-Type`. Each part has its own headers (`Content-Disposition`, and for files a `filename` and `Content-Type`).

**Why a file can't ride urlencoded:** urlencoded is a **text** format — raw file bytes would have to be Base64-encoded (about **+33% size**) and there is no place to carry the filename or MIME type. Multipart preserves the bytes exactly and wraps them with metadata. Hence: **files require `multipart/form-data`**.

| | urlencoded | multipart |
|---|---|---|
| `Content-Type` | `application/x-www-form-urlencoded` | `multipart/form-data; boundary=…` |
| Body shape | `a=1&b=2` (percent-encoded) | parts separated by a boundary |
| Files | ❌ no | ✅ yes (raw bytes + filename) |
| Overhead | minimal | boundary + per-part headers |
| Default for forms | ✅ yes | only when set |

---

### 4. Client-Side Validation

HTML attributes give free, declarative validation:

```html
<input type="text" name="user" required minlength="3" maxlength="20" pattern="[a-zA-Z0-9_]+">
<input type="number" name="age" min="18" max="120">
<input type="email" name="email" required>
```

The browser's **Constraint Validation API** lets JavaScript query and extend it:

```js
const input = document.querySelector('#user');
input.checkValidity();      // false if constraints violated
input.validity.tooShort;    // fine-grained reason flags
input.validationMessage;    // human-readable message
input.setCustomValidity('That username is taken'); // force an error
input.setCustomValidity('');                        // clear it
```

- `form.checkValidity()` validates every control and returns a boolean.
- **`novalidate`** on the `<form>` disables the browser's automatic blocking so *you* control submission — then call `reportValidity()` yourself to show the native bubbles.
- `setCustomValidity` is how you keep a server rule (e.g. "username taken", checked via `fetch`) in the same UI.

**Why client validation is UX, not security:** the attributes are just text in the HTML the user received. A user can edit them in DevTools, disable JS, or send a `curl` request with `age=5`. The browser check only saves a round-trip; the server must re-validate the same rules or the data is untrusted. This is the boundary between "nice form" and "secure form" (see the security note in §1).

---

### 5. The FormData API

`FormData` is the object that represents a multipart payload in JavaScript.

```html
<form id="signup">
  <input name="email" value="a@b.com">
  <input type="file" name="avatar" accept="image/*">
</form>
```

```js
const form = document.querySelector('#signup');

// 1. Auto-collect every named control in the form:
const fd = new FormData(form);

// 2. Or build it by hand:
fd.append('plan', 'pro');
const file = form.querySelector('input[type=file]').files[0];
fd.append('avatar', file);           // a File or Blob — binary allowed

// 3. Read / inspect:
fd.get('email');                     // "a@b.com"
fd.has('avatar');                    // true
fd.delete('plan');
for (const [key, value] of fd.entries()) console.log(key, value);
```

- `new FormData(formElement)` is the shortcut: every control **with a `name`** is included automatically, files and all.
- A file from `<input type="file">` is a **`File`** object (a `Blob` with a name). You can only append a `File`/`Blob`; you cannot read the file's bytes from the input by design (browser security) — you use `FileReader` or an object URL for that.
- The `accept` attribute (`accept="image/*"`, `accept=".pdf,.docx"`) filters the picker; it is a hint, not enforcement.
- **Drag & drop basics:** listen for `dragover` (call `preventDefault()` so the drop is allowed), then `drop`, and read `e.dataTransfer.files` — the same `File` objects.

```js
dropZone.addEventListener('dragover', (e) => e.preventDefault());
dropZone.addEventListener('drop', (e) => {
  e.preventDefault();
  const file = e.dataTransfer.files[0];
  if (file) handleFile(file);
});
```

---

### 6. Fetch

`fetch` returns a **Promise** that resolves once the response headers arrive. Three shapes cover everything in this course.

**GET with query parameters** — data in the URL, matching a `method="get"` form:

```js
const params = new URLSearchParams({ q: 'shoes', page: '2' });
const res = await fetch(`/search?${params}`);   // → /search?q=shoes&page=2
const json = await res.json();
```

**POST with JSON** — a real API body, not a form:

```js
const res = await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },   // WE set this
  body: JSON.stringify({ name: 'Alex', age: 21 }),
});
```

**POST with FormData** — files and form fields:

```js
const res = await fetch('/upload', {
  method: 'POST',
  body: fd,                    // FormData: browser sets Content-Type + boundary
  // ⛔ do NOT add headers here
});
```

**Reading the response and telling errors apart:**

```js
async function send(fd) {
  let res;
  try {
    res = await fetch('/upload', { method: 'POST', body: fd });
  } catch (err) {
    // A NETWORK failure: offline, DNS, CORS, server down.
    console.error('network error:', err);
    return;
  }
  // fetch RESOLVED — even for 404/500. Check status yourself:
  if (!res.ok) {
    console.error('HTTP error:', res.status, await res.text());
    return;
  }
  const data = await res.json();
  console.log('ok:', data);
}
```

- `res.ok` is `true` for status **200–299**; otherwise it is `false` even though the Promise *resolved*.
- `res.json()` / `res.text()` / `res.blob()` are themselves async — **`await` them**.
- `try/catch` inside an `async` function catches both network rejections and *your* thrown errors; it does **not** catch a 500 (that is a resolved response). Check `res.ok` separately.

**The options object is worth reading once, properly.** The second argument to `fetch` is a `RequestInit`; these are the keys you will actually use:

| Option | Purpose | Typical value |
|---|---|---|
| `method` | HTTP verb | `"GET"`, `"POST"`, `"PUT"`, `"DELETE"` |
| `headers` | request headers | `{ "Content-Type": "application/json" }` |
| `body` | payload (**not allowed on GET/HEAD**) | `JSON.stringify(x)` or a `FormData` |
| `credentials` | send cookies cross-origin | `"same-origin"` (default), `"include"` |
| `signal` | cancel via an `AbortController` | `controller.signal` |
| `mode` | CORS behavior | `"cors"` (default cross-origin), `"same-origin"` |

**Status codes tell you *which* error.** `res.ok` is a coarse boolean; the code is the diagnosis. `200 OK` / `201 Created` mean success. `400 Bad Request` means your payload was malformed or failed validation. `401`/`403` are authentication vs authorization. `413 Payload Too Large` is the "file too big" your server enforces. `415 Unsupported Media Type` means the server rejected your `Content-Type` (often a wrong `enctype`). Log `res.status` — it is the fastest clue you will get.

**Classic submit vs `fetch` — when to reach for which:**

| | Classic form submit | `fetch` |
|---|---|---|
| Page behavior | navigates to the response | stays on the page |
| Result handling | the server renders the next page | JS reads JSON/text and updates the DOM |
| Works without JS | ✅ yes | ❌ no |
| File upload | ✅ best fit | ✅ via `FormData` |
| Progress feedback | via navigation only | can show spinner / preview |
| Best for | login, simple CRUD, accessibility-first pages | live search, autosave, previews, AJAX |

The mature choice is often **classic submit as the baseline, `fetch` layered on top** for the fast paths — that is progressive enhancement, and it is the answer to a later open question in this note.

---

### 7. Upload Preview Example (HTML + JS)

```html
<input type="file" id="picker" accept="image/*">
<img id="preview" alt="preview" width="160" hidden>
<button id="send" disabled>Upload</button>
<pre id="out"></pre>

<script type="module">
  const picker  = document.querySelector('#picker');
  const preview = document.querySelector('#preview');
  const send    = document.querySelector('#send');
  const out     = document.querySelector('#out');

  let chosen = null;

  picker.addEventListener('change', () => {
    const file = picker.files[0];
    if (!file) return;
    chosen = file;

    preview.src = URL.createObjectURL(file);  // local preview, no upload yet
    preview.hidden = false;
    send.disabled = false;
  });

  send.addEventListener('click', async () => {
    if (!chosen) return;

    const formData = new FormData();
    formData.append('avatar', chosen);        // File → multipart part
    formData.append('title', 'My avatar');

    out.textContent = 'Uploading…';
    try {
      const res = await fetch('/upload', { method: 'POST', body: formData });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      out.textContent = JSON.stringify(await res.json(), null, 2);
    } catch (err) {
      out.textContent = `Upload failed: ${err.message}`;
    } finally {
      URL.revokeObjectURL(preview.src);       // free the object URL
    }
  });
</script>
```

- `URL.createObjectURL(file)` creates a temporary `blob:` URL pointing at the in-memory file — instant preview with **no network**. (`FileReader.readAsDataURL` is the older alternative, producing a Base64 `data:` URL.) **Revoke it** when done to release memory.
- The actual bytes leave the machine only at `fetch('/upload', { body: formData })`. Watch the request in DevTools → Network: `Content-Type: multipart/form-data; boundary=…` is set **by the browser**.
- The `POST /upload` handler (Express + `multer`) is in [`../../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md).

---

## 🧾 Command / Code Cheat Sheet

| Construct | What it does | When to use | Example |
|---|---|---|---|
| `<form action method enctype>` | declares where/how the request goes | every data submission | `method="post" enctype="multipart/form-data"` |
| `name="email"` | the key the server receives | every submitted control | without it, the field is dropped |
| `<label for="id">` | links text to control (click + a11y) | every input | `<label for="qty">Qty</label>` |
| `type="file" accept="image/*"` | file picker filtered by type | uploads | `<input type="file" name="avatar">` |
| `required min max pattern` | declarative validation | quick UX rules | `<input required minlength="3">` |
| `input.checkValidity()` / `.setCustomValidity()` | query / extend the Constraint Validation API | custom rules | `setCustomValidity('taken')` |
| `novalidate` | stop the browser from auto-blocking submit | you want JS-controlled submit | `<form novalidate>` |
| `new FormData(form)` | collect all named controls | serialize a form in JS | `const fd = new FormData(form)` |
| `fd.append(name, file)` | add a field/file (binary allowed) | build multipart by hand | `fd.append('avatar', file)` |
| `URLSearchParams` | safely build a query string | GET requests | `new URLSearchParams({q:'a b'})` |
| `fetch(url, {method, headers, body})` | HTTP request → Promise<Response> | any JS request | `await fetch('/api')` |
| `res.ok` / `res.status` | detect HTTP errors (fetch won't) | always, after `fetch` | `if (!res.ok) …` |
| `URL.createObjectURL(file)` | local preview of a File | image preview before upload | `preview.src = URL.createObjectURL(f)` |
| `FileReader.readAsDataURL(file)` | read file as Base64 data URL | preview / small text files | `reader.readAsDataURL(file)` |
| `e.dataTransfer.files` | files from a drop event | drag & drop upload | `dropZone.addEventListener('drop', …)` |

---

## 🧪 Practical Lab

> Run these against a local Express app (`node app.js`) — the server half is in [`../../web_server/08-`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md). Open DevTools → Network for every step.

**Lab 1 — Intercept a submit and log the wire format**

```html
<form id="f" method="post" action="/subscribe">
  <input name="email" value="a@b.com">
  <input name="plan" value="pro">
  <button type="submit">Send</button>
</form>
<script>
  document.querySelector('#f').addEventListener('submit', (e) => {
    e.preventDefault();                              // stop classic navigation
    const fd = new FormData(e.target);
    console.log('method:', e.target.method);         // "post"
    console.log('action:', e.target.action);
    console.log('pairs:', [...fd.entries()]);         // [["email","a@b.com"],["plan","pro"]]
    console.log('body:', new URLSearchParams(fd).toString()); // email=a%40b.com&plan=pro
  });
</script>
```

**Expected console output**
```text
method: post
action: http://localhost:3000/subscribe
pairs: [["email","a@b.com"],["plan","pro"]]
body: email=a%40b.com&plan=pro
```
*What to notice:* `e.preventDefault()` changes submission from "navigate away" to "stays on page"; `[...fd.entries()]` shows exactly what the server would have received; `URLSearchParams(fd)` reproduces the urlencoded body.

**Lab 2 — Upload preview flow**

Use the §7 snippet. Steps:

1. Pick a PNG. **Expected:** the `<img>` appears immediately with your image; the **Network tab shows no request yet** (the preview is local, via `blob:`).
2. Click **Upload**. **Expected:** a `POST /upload` appears in Network with `Content-Type: multipart/form-data; boundary=…`; request payload shows two parts (`avatar`, `title`); the `<pre>` prints the JSON response (e.g. `{"ok":true,"url":"/uploads/me.png"}`).
3. Stop the server and click **Upload** again. **Expected:** console shows `Upload failed: …` (network/fetch error) — *not* an HTTP error, because fetch rejected.
4. Return a `500` from the server (temporarily `res.status(500)`). **Expected:** fetch **resolves**, `res.ok` is `false`, and your code throws `HTTP 500` itself.

*What to notice:* steps 3 and 4 are the two different failure classes — rejected Promise vs resolved-but-not-ok — and your code must handle both.

---

## ⚠️ Common Pitfalls & Gotchas

| Pitfall | Symptom | Fix |
|---|---|---|
| Missing `name` attribute | server sees `undefined`/empty for that field | add `name="…"`; only named controls submit |
| Wrong `enctype` for a file | file arrives as a string or not at all | set `enctype="multipart/form-data"` (or use `FormData`) |
| Hand-setting `Content-Type` with `FormData` | `multipart: Boundary not found` on the server | omit the header; let the browser set the boundary |
| Trusting client-side validation | bad data in the DB despite "required" | re-validate on the server (see security/01) |
| Reading input value before it's filled / before submit | empty or stale data | read inside the `submit`/`change` handler, not at load |
| Forgetting `await` before `res.json()` / `fetch()` | you log a `Promise`, not the data | `const data = await res.json()` |
| Assuming `fetch` rejects on `404`/`500` | error silently treated as success | check `res.ok` / `res.status` |
| Not calling `e.preventDefault()` | page reloads and you lose the JS flow | call it first in the `submit` listener |
| `FormData` includes disabled controls? | disabled field missing from payload | disabled controls are **not** submitted — by design |
| Never revoking object URLs | memory grows on repeated previews | `URL.revokeObjectURL(url)` when done |

---

## 🧪 Practice Tasks

> No solutions. Build them, then ask the AI to review your Network-tab evidence.

- [ ] **Task 1 — Serialize and compare.** Build a form with a text field, a checkbox group, a select, and a textarea. On submit, `preventDefault()` and log: the `FormData` entries, the urlencoded body (`URLSearchParams`), and a JSON object. Explain why the checkbox group produces zero, one, or many entries.
- [ ] **Task 2 — Custom validation with the API.** Build a username field that calls `setCustomValidity('taken')` when a `fetch` GET to `/api/check?user=…` returns `{taken:true}`, and clears it otherwise. Prevent submit while invalid.
- [ ] **Task 3 — Drag & drop upload.** Extend the §7 preview to accept a dropped image (`dragover`/`drop`, `preventDefault`, `dataTransfer.files`) and reject non-images via `file.type`. Show the file name and size before upload.
- [ ] **Task 4 — Error taxonomy.** Point a `fetch POST` at (a) a route returning `400`, (b) a route returning `500`, and (c) a stopped server. For each, classify whether the Promise *rejected* or *resolved*, and write the exact `catch`/`res.ok` branch that fires. Explain the difference in one sentence each.

## 🔗 Tech Cross-References

- **JavaScript async & the event loop** — `fetch` is Promise-based; `await` is the same microtask mechanism as note 04/05; the `submit` listener is an event-loop callback. [`04-javascript-language-core.md`](04-javascript-language-core.md) · [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md)
- **Express routing (server half)** — the request your form/fetch produces lands in an Express route; method + path matching happens there. [`../../web_server/04-routing-in-express.md`](../web_server/04-routing-in-express.md)
- **Upload handling (server half)** — `multer` parses the multipart body this note builds, validates and stores the file. [`../../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md)
- **Web security** — client validation is cosmetic; injection/XSS/CSRF defenses belong on the server. [`../../security/01-web-security-injection-xss-csrf-session-hijacking.md`](../security/01-web-security-injection-xss-csrf-session-hijacking.md)
- **Databases** — the row created by the server after this request is validated and inserted here. [`../../databases/README.md`](../databases/README.md)

## 📝 Research Journey

- **Why:** The course project's final feature is a file upload with an image preview, and everything I had written before was one-directional — the server rendered HTML *to* me. This note is the first time data travels *from* me to the server (other than a login form), so it is the moment the request/response loop became bidirectional.
- **Struggle:** The `Content-Type` boundary cost me the most time. I set `'Content-Type': 'multipart/form-data'` out of habit and the server replied `Multipart: Boundary not found`. It took reading an actual raw request in DevTools to see that the browser appends `; boundary=…` and that by setting the header I had erased it.
- **Aha moment:** Opening the Network tab and reading the literal body — `email=a%40b.com&plan=pro` for a normal form, and the `------WebKitFormBoundary…` parts for a file — turned "encodings" from vocabulary into something I could *see*. After that, debugging submissions stopped being guesswork.
- **Career link:** Any AI product with an economics/business use case eventually ingests user data — a CSV of transactions, a photo of a receipt, a document to summarize. That intake is exactly this note: a form, a `FormData`, a `POST`, and a server that validates before it trusts. Getting the wire format right is what makes the data pipeline possible.

## 🔗 Related Topics

- [`04-javascript-language-core.md`](04-javascript-language-core.md) — `async`/`await`, Promises, and the coercion rules behind `URLSearchParams`.
- [`05-dom-and-browser-event-loop.md`](05-dom-and-browser-event-loop.md) — the `submit`/`change`/`drop` events and how async callbacks are scheduled.
- [`01-html-structure-and-semantics.md`](01-html-structure-and-semantics.md) — the form tags, labels, and accessibility covered here in more depth.
- [`../../web_server/08-advanced-features-pagination-wysiwyg-upload.md`](../web_server/08-advanced-features-pagination-wysiwyg-upload.md) — the Express/multer server that answers `POST /upload`.
- [SUGGESTED] `09-client-state-and-storage.md` — because after you receive a response you must decide where it lives: DOM, `localStorage`, or `sessionStorage`; a natural follow-on to note 06 in `web_server/`.
- [SUGGESTED] `bridge-postgrest-and-fetch.md` — bridge note: the same `fetch` call against an auto-generated PostgREST endpoint, connecting this note to `databases/`.

## 🤔 Open Questions

- [ ] When should a form fall back to a normal (non-AJAX) submission so the page still works without JavaScript — and how do we build that progressive enhancement cleanly in Express?
- [ ] What is the practical difference between `URL.createObjectURL` and `FileReader.readAsDataURL` for large images — memory, revocation, and rendering?
- [ ] How does CORS intersect with a `fetch` upload when the API is on a different origin than the SSR app?
- [ ] Where exactly should file size/type limits live — the `accept` attribute, a JS check before upload, the Express/multer config, or all three — and why is the server one the only trustworthy limit?

---
*The page can lie. The wire cannot. When in doubt, open the Network tab and read the request.*
