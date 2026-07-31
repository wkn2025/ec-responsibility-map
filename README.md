# EC Responsibility Map

An interactive map of who is responsible for every task in the operation, and where the SOP
for that task lives.

- **Search list** — the default view. Find your task, open its SOP, two clicks.
- **Radial map** — a toggle, for seeing ownership and documentation gaps across the whole operation.

Static site. No server, no database, no build step. Two files do the work: `index.html` and `data.json`.

---

## Before real data goes in — read this

GitHub Pages is a **public** host. On Free and Pro plans a Pages site is reachable by anyone with
the URL **even when the repository is private**. Serving Pages with access control requires GitHub
Enterprise Cloud. There is no login you can add to a static site that is actually a security control.

This site holds staff names, their responsibilities and the internal systems they use. That is
HR-adjacent internal data. `robots.txt` and `noindex` are in the repo, but they only reduce
discovery — they are **not** access control.

Pick one before publishing anything real:

1. **Enterprise Pages with access control** — the only option that puts real names on a public host legitimately.
2. **De-identify the public deploy** — publish initials or employee IDs instead of full names.
   The app is already built for this: ownership is stored as `ownerId` and the display name comes
   from the `people` lookup. Swapping `displayName` to initials is a **data** change, not a code change,
   and every derived stat keeps working.
3. **Prototype only** — keep Pages for the concept demo with fabricated sample data, host the real
   thing internally.

**The `data.json` in this repo today is fabricated sample data. No real staff names.**

---

## Publishing a change — the routine

You do not need to know git. You need to do four things.

### 1. Unlock editing

Open the site with `?edit=1` on the end of the URL:

```
https://<your-org>.github.io/<repo>/?edit=1
```

Or press **Ctrl + Shift + E** on the page. A status bar appears at the top.

> This is a convenience for a single editor. It is not a login and it is not security.
> Anyone who can reach the URL can turn it on. See the section above.

### 2. Edit

Change what you need. Every keystroke is saved **as a draft in your browser** — it survives a
refresh, a browser restart and a crash. The status bar says `4 unsaved changes — not yet published`
in lime while drafts exist.

**Drafts are not live.** Nobody else can see them. Nothing is live until step 4.

`Review changes` shows exactly what you altered: task, field, old value, new value.
`Discard all drafts` throws them away.

### 3. Publish → download the file

Press **Publish**, type your name, press **Download data.json**. A file lands in your Downloads folder.

### 4. Put the file in the repo

Either:

**On github.com (no git needed):**
1. Go to the repository front page.
2. Click **Add file → Upload files**.
3. Drag the downloaded `data.json` in. It replaces the old one.
4. In the commit message box, paste the message the app suggested, e.g.
   `data: 4 changes to Billing (Tiken, 12 Aug)`
5. Click **Commit changes**.

**Or from a terminal:**

```bash
git add data.json && git commit -m "data: 4 changes to Billing (Tiken, 12 Aug)" && git push
```

GitHub Pages redeploys in under a minute. Reload the site. It sees the higher version number and
clears your drafts by itself. The status bar goes back to `v15 · updated 12 Aug 2026`.

### Optional: commit straight from the browser

The Publish dialog also offers **Commit from browser…**. It uses the GitHub Contents API to write
`data.json` directly, so you skip steps 3 and 4.

Conditions — these are not negotiable:

- Use a **fine-grained personal access token**, scoped to **this one repository**, permission
  **Contents: read and write**, nothing else, with a short expiry.
- The token lives in `sessionStorage` and dies when you close the tab. There is a **Forget token** button.
- **Never** put the token in a file in this repo. Anything committed to a public repo is world-readable
  forever, including in git history — deleting it later does not help.
- If the commit fails for any reason, the app falls back to downloading the file so your work is
  never trapped in the browser.

---

## If something goes wrong

**"Data could not be loaded"** — `data.json` is missing, is not valid JSON, or Pages has not finished
redeploying. The app deliberately has **no built-in sample data**: it will never invent rows to
paper over a load failure, because fake ownership data that looks real is worse than a visible error.
Wait a minute and reload. If it persists, check the file on github.com — the JSON view will point at
the syntax error.

**"The published data has changed since your drafts were made"** — you edited from a second device,
or someone else published. Nothing is merged silently. Choose `Keep my drafts` or `Discard my drafts`.

**I broke the data and I do not use git** — in edit mode with no pending changes, the status bar
offers `Download previous version`: the last three files this browser produced. Download one and
upload it as `data.json`.

**I broke the data and I do use git** — the repository is the real audit trail. Every publish is a
commit with an author, a timestamp and a diff. `git revert` any of them, or restore an old file from
the commit history on github.com.

**Data issues (n)** in the footer — rows the loader could not use as written (missing Department or
Task, duplicate ids, an `ownerId` with no matching person, an unrecognised Automation value). Click it
for the list. Nothing was invented to cover them.

---

## The columns

`data.json` is one object with `people` and `tasks`.

### Tasks

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | Stable id, generated once and never reused. Every stat, draft and diff keys off it. **Never renumber these** — matching on task name breaks the first time something is renamed. |
| `department` | yes | Level 1. Becomes a hub on the map. |
| `subFunction` | yes | Level 2. Groups tasks inside a department. |
| `task` | yes | Level 3. The leaf. |
| `ownerId` | — | Points at a `people` entry. `null` means unassigned — the task shows grey on the map. |
| `sopLink` | — | URL to SharePoint or a network share. **Empty is meaningful**: it makes the task lime on the map and puts a `No SOP` chip in the list. |
| `description` | — | One short paragraph. |
| `systems` | — | Array of strings, e.g. `["Globo","Citi"]`. Feeds the "systems required" onboarding list in the person view. |
| `automation` | — | `Manual`, `Semi` or `Auto`. |
| `source` | — | `System` or `Manual`. `System` rows show a note that a future automated export will overwrite edits. Display only for now. |
| `workTypeId` | — | Stable id from the claims system. Reserved for the future join. Not used yet. |
| `deleted` | — | `true` hides the row. Deleting is soft on purpose — the id must stay stable for diffs to make sense. |

### People

| Field | Meaning |
|---|---|
| `id` | Stable id. This is what tasks point at. |
| `displayName` | What the app shows. **Change this one field to de-identify the whole site.** |
| `initials` | Optional. |
| `role` | Optional. Shown in the person view header. |

The Department / Sub-function / Owner / Systems dropdowns in the editor are built from values already
in the data, each with an explicit `+ Add new`. That is the only thing standing between you and
`Back Office` / `Backoffice` / `BO` existing as three departments. Renaming a department offers to
update every row that uses the old name in one action. Renaming a person needs no fan-out at all —
tasks store `ownerId`, so every owned task follows automatically.

The three-level hierarchy is **derived** from the `department` and `subFunction` columns. There is no
nested structure to hand-maintain.

---

## Repo contents

```
index.html            the whole app — both views, editing included
data.json             the data. Code and data never share a file.
vendor/three.min.js   Three.js r160, vendored. Loaded only when the map is first opened.
fonts/*.woff2         Archivo (variable width), Public Sans, IBM Plex Mono. No Google Fonts at runtime.
.nojekyll             stops GitHub Pages' Jekyll step mangling paths
robots.txt            Disallow: / — hygiene, not access control
README.md             this file
```

### Brand colours

The three brand values are taken from the Euro-Center logo SVG on euro-center.com, not
eyeballed. The site's own stylesheet agrees with them (`--bs-primary`, `--bs-green`, and the
navy used across the main navigation):

| Token | Value | Role here |
|---|---|---|
| `--ec-navy` | `#283583` | Structural — department hubs, links, pressed states |
| `--ec-blue` | `#0084CC` | Documented — a task with an owner *and* an SOP |
| `--ec-lime` | `#CFCE22` | **Missing SOP.** The only alarm colour in the app |

Everything else (`--void`, `--void-lift`, `--mist`, `--paper`, `--grey-node`) is a support tone
derived on the navy's hue, 231°, so the dark field reads as the same family rather than as a
generic dark theme.

Two things worth knowing before you change any of this:

- **All colour lives in the `:root` block of `index.html`.** The WebGL map reads those custom
  properties at boot rather than keeping its own copies, so re-colouring the whole app — list,
  panels and map nodes alike — is an edit to that one block. There are no colour literals
  anywhere else in the file.
- **Filled blue buttons use `--ec-blue-deep` (`#006AA3`), not the brand blue.** White on
  `#0084CC` is 4.06:1, below the 4.5:1 WCAG AA floor for normal text. `#006AA3` is
  Euro-Center's own darker blue (their `--bs-link-hover-color`) and gives 5.85:1. The brand
  blue is untouched everywhere it is a mark, a border or a text link. Every text/background
  pair in the app now passes AA; the weakest is the blue mark on the dark field at 4.84:1.

One deliberate divergence: on euro-center.com the lime is the **call-to-action** colour
(`.green_button`). Here it means **undocumented**. That is the point of the whole map — lime is
the loudest thing on a navy field, so documentation debt is impossible to miss. If that clash
ever matters for a leadership demo, change the meaning in one place (`nodeColour()`), not the
brand value.

### A note on the vendored Three.js

`vendor/three.min.js` is **r160**, the classic UMD build that defines a global `THREE`. It works, and
it is loaded with a plain `<script>` tag only when the map is first opened — the list view never
fetches it. On load it logs one console warning saying UMD builds are deprecated; that is expected
and harmless.

r160 is the **last release that ships this build**. If you ever upgrade Three.js, you cannot just drop
a newer file in: newer releases are ES modules only, so `index.html` would need an import map or a
bundler. For an MVP that must run with no build step, staying on r160 is the deliberate choice.

### Why data lives in its own file

Deploying new code must not be able to touch the data. There is no seed data inside `index.html` —
not as a fallback, not as a default, not commented out — because if sample data lives in the code
file, a future code deploy can resurrect it over real data.

`data.json` carries a `schemaVersion`. If the file's version is lower than the code expects, the app
migrates it **in memory** and leaves the file alone until the next publish. New code always reads old data.

The fetch is always cache-busted (`./data.json?t=<timestamp>`, `cache: 'no-store'`). GitHub Pages sits
behind a CDN with roughly a 10-minute cache — without this you would commit a change, reload, see the
old data and conclude the publish failed.

---

## Deploying

1. Push this repo to GitHub.
2. **Settings → Pages → Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
3. Wait for the first build. The URL appears on the same settings page.

Nothing to install, nothing to build.

## Local preview

`fetch` needs a real origin, so open it through a server rather than double-clicking the file:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000/`.

---

## Notes on scope

Deliberately **not** built: any live integration with the claims system, notifications or overdue
alerts, file uploads (SOPs are links only, so SharePoint permissions are inherited for free), review
cadence or staleness flags, multi-user editing, real authentication, and any performance or
productivity metric.

That last one is a design rule, not an oversight. Every label frames the data as **coverage and
continuity** — never productivity. No scores, no rankings, no leaderboard. The moment people believe
this feeds appraisals, they start under-declaring what they do and the data dies.

Two person-view stats are deferred because the MVP does not have the data for them: *backs up for*
(backup is a live out-of-office state in the claims system, not a static attribute) and *stale tasks*
(needs a review-date field, which is deliberately out of scope).
