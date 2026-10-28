# আবেদন · Application Portal

One page, three doors — **student admission**, **teacher application**, and **application status**.
It is a single self-contained file: no build step, no npm, no images, no external CSS or JS.
Upload `index.html` to its own GitHub repository and it is live.

```
apply-portal/
└── index.html      ← the whole portal (HTML + CSS + JS in one file)
```

---

## 1. Publish it (5 minutes)

1. Create a new repository, e.g. **`apply-portal`** (any name works — see §3 if you use another).
2. Upload **`index.html`** so it sits at the repository **root** (not inside a sub-folder).
3. Repository → **Settings → Pages** → *Source*: **Deploy from a branch** → branch **`main`**, folder **`/ (root)`** → Save.
4. Wait a minute, then open:

   ```
   https://<your-user>.github.io/apply-portal/
   ```

No `.nojekyll`, no workflow, no Actions — this is plain HTML.

---

## 2. The only two things you may need to edit

Both are `<a class="door" href="…">` values inside `index.html`, marked with a comment:

| Door | Where | Current value |
|---|---|---|
| শিক্ষার্থী আবেদন · Student | `<!-- 01 · STUDENT  ← EDIT … -->` | `https://jamiaislamia.github.io/admission-form/` |
| শিক্ষক আবেদন · Teacher | `<!-- 02 · TEACHER  ← EDIT … -->` | `https://jamiaislamia.github.io/teacher-form/` |

If your teacher-form repository has a different name, change that one line to its real Pages URL
(e.g. `https://jamiaislamia.github.io/teacher-application/`). The third door (`href="#track"`)
needs no editing — it scrolls to the checker on the same page.

The footer link back to the website is `https://jamiaislamia.github.io/` — change it if the site
moves.

---

## 3. How the tracker works

The status checker talks **directly to the institute's Firebase Realtime Database**, read-only:

| ID you type | Node it reads | Who writes it |
|---|---|---|
| `APP-XXXXXXXX` | `/admissions` | the student admission form |
| `TCH-XXXXXXXX` | `/teacher-applications` | the teacher application form |

* It sends **GET** requests only — the page can never change or delete a record.
* If the prefix does not match the node, the other node is checked as a fallback, so an ID still
  resolves if it was filed under the unexpected kind.
* No API key, no SDK, no login: the database URL is the public REST endpoint
  `https://sms-admission-4f632-default-rtdb.asia-southeast1.firebasedatabase.app`.

**One requirement:** the database rules must allow public **read** on those two nodes. The project's
`database.rules.json` currently declares:

```json
"admissions":           { ".read": true, ".write": true },
"teacher-applications": { ".read": true, ".write": true }
```

If you ever tighten `".read"`, the checker will report
“সার্ভারে সংযোগ করা যায়নি · Could not reach the server”.

Status tags map to the records the admin panel writes:
`pending` → amber *পর্যালোচনাধীন*, `approved` → green *অনুমোদিত*,
`declined` → red *প্রত্যাখ্যাত*, `waitlisted` → amber *অপেক্ষমাণ*.

---

## 4. Preview locally

Double-click `index.html` — it works from `file://` too (the lookup needs internet).
Or, from this folder:

```
python -m http.server 8080
# then open http://localhost:8080/
```

---

## 5. Design notes

* Same family as the admission and teacher forms: navy `#0a0e1a` surfaces, indigo `#5c6ef8`
  accent, Cormorant Garamond headings (Bangla falls through to Hind Siliguri), DM Mono for every
  micro-label.
* One accent per screen; semantic colour appears only on status tags and result panels
  (teal = approved, amber = pending/waitlisted, crimson = declined/error).
* Cards keep the signature 3px accent bar (40px stub → full width on hover).
* Hover, focus, `prefers-reduced-motion` and keyboard `Enter` are all handled; the page is usable
  at 320px wide.

---

## 6. Tested

`node _test_apply_portal.js` (in the project root) loads this exact file in a headless DOM and
asserts 27 behaviours: the three doors and their targets, no external dependencies, teacher and
student lookups, approved/pending tags, unknown and malformed IDs, the offline panel, and that
every request is a GET.

```
RESULT: 27 passed, 0 failed
```
