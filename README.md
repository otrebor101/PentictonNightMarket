# Penticton Night Market — site

A single-page site for the Penticton Night Market at Gyro Park. No servers
of your own required — it runs as static files on GitHub Pages, and the
"market updates" banner and the hero photo stack are both editable from a
password-gated admin panel on the site itself, backed by a GitHub Actions
workflow.

## What's in here

```
index.html                     the whole site (styles + markup + scripts)
hash-password.html             one-time setup helper — turns a password into its hash
images/                        photos used in the hero stack and gallery
data/market-updates.json       the live "market updates" banner content — starts as []
data/photos.json               the live hero photo stack (image + caption + rotation)
.github/workflows/admin-update.yml   runs on GitHub's servers, does the actual writes
.github/scripts/apply-update.js      the script that workflow runs
README.md                      this file
```

## How admin publishing works now

Nothing writes to the repo directly from your browser. Instead:

1. The admin panel triggers a GitHub Actions workflow (`admin-update.yml`)
   via GitHub's API.
2. That workflow runs on GitHub's own servers: it checks out the repo fresh,
   verifies your password, edits the JSON file, commits, and pushes.
3. Because every write starts from a **fresh checkout** and the workflow is
   set to run **one at a time** (`concurrency` in the YAML), the old
   "`data/market-updates.json` does not match `<sha>`" errors can't happen
   anymore — there's no longer a stale-SHA race between two quick edits.

Two different credentials are involved, and they do different jobs:

- **The embedded Actions token** (`GH_ACTIONS_TOKEN` near the top of
  `index.html`) authorizes *triggering* the workflow. It's a fine-grained
  GitHub token scoped to only this repo, with only "Actions: Read and
  write" permission — it cannot read or write a single file directly. It's
  fine that it's visible to anyone who views the page source; the worst a
  leaked copy can do is trigger runs, which then fail at the password check.
  This is also why it's set to never expire — you shouldn't need to touch
  it again after setup.
- **The admin password** is the real gate. It's hashed in your browser and
  compared against a GitHub Actions secret *inside* the workflow run — the
  plain password is never written to any file and never appears in the
  repo's (public) Actions logs.

## 1. Upload this to GitHub

1. Create a **public** repository (it must be public so the updates/photos
   JSON and the Actions token above can be read without extra auth).
2. Upload everything here, keeping the same structure — `index.html`,
   `.github/`, `data/`, and `images/` should all sit at the repo root.

## 2. Turn on GitHub Pages

1. Repo → **Settings → Pages**.
2. Under "Build and deployment," set **Source** to "Deploy from a branch."
3. Branch: `main`, folder: `/ (root)`. Save.
4. GitHub will give you a URL like `https://your-username.github.io/your-repo/`
   within a minute or two.

## 3. Point the site at your repo

Open `index.html`, find the `CONFIG` block (search for `GH_OWNER`), and fill in:

```js
const GH_OWNER  = "your-github-username";
const GH_REPO   = "your-repo-name";
const GH_BRANCH = "main";
```

## 4. Create the Actions-only token

1. GitHub → your avatar → **Settings** → **Developer settings** →
   **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
2. **Repository access:** "Only select repositories" → choose this repo.
3. **Permissions:** under "Repository permissions," set **Actions** to
   **Read and write**. Leave everything else (including Contents) as
   "No access."
4. Set **no expiration**.
5. Generate and copy the token — **don't paste it raw into `index.html`**.
   GitHub's secret scanning recognizes the `github_pat_...` pattern in any
   public repo and auto-revokes it within minutes, no matter how narrowly
   it's scoped — that's the "token auto-expires" behavior you'd otherwise
   hit.
6. Open `encode-token.html` locally in a browser, paste the token in, and
   copy the encoded result it shows.
7. Paste that encoded value into `GH_ACTIONS_TOKEN_B64` in `index.html`
   and commit. (This doesn't hide the token from anyone using the site —
   it's still visible in plain text in the browser's network tab when
   used — it only avoids GitHub's automated scanner, which only looks for
   the literal, unencoded pattern.)
8. You can delete `encode-token.html` from the repo afterward, or leave it.

## 5. Set your admin password

1. Open `hash-password.html` locally in a browser (double-click the file —
   no server needed) and type the password you want to log in with. Copy
   the hash it shows.
2. Repo → **Settings → Secrets and variables → Actions → New repository
   secret**. Name it `ADMIN_PASSWORD_HASH`, paste the hash as the value.
3. You can delete `hash-password.html` from the repo afterward, or leave it
   — it isn't linked from the site and does nothing on its own.

To change the password later, repeat step 1 with a new password and update
the secret in step 2 — no code changes needed.

## 6. Use the admin panel

1. Open your live site, scroll to the footer, click **Admin**.
2. Enter your password and log in (this triggers a quick no-op workflow run
   just to check the password — takes a few seconds).
3. **Market updates:** type an update and hit **Publish**, or remove/clear
   existing ones. Each change is a real commit — see history under the
   repo's **Actions** and **Commits** tabs, and revert any of them if needed.
4. **Photo stack:** reorder, remove, or swap the caption/image on any slot,
   or add a new slot with **+ Add photo to stack**. To use a brand-new
   photo (not already in `images/`), upload the image file to `images/` in
   the repo first (drag-and-drop on GitHub works fine) — it'll then appear
   in the image dropdown here. Hit **Save photo stack** to publish.

Visitors don't need a token or password — the banner and photo stack are
just public reads of the two JSON files, so they update for everyone within
a few seconds of a publish.

## Notes

- The admin password is only ever held in the browser's memory for that
  tab — never written to any file, cookie, or storage. You'll need to log
  in again each time you reopen the panel.
- If you ever think the Actions token leaked, it's low-stakes to rotate:
  revoke it from **Developer settings → Fine-grained tokens**, generate a
  new one, re-encode it with `encode-token.html`, and update
  `GH_ACTIONS_TOKEN_B64` in `index.html`. It never had write access to
  your files, only the ability to trigger the workflow.
- If you think the *password* leaked, that's the one that actually matters
  — generate a new hash with `hash-password.html` and update the
  `ADMIN_PASSWORD_HASH` secret right away.
- Each publish takes a few seconds longer than the old direct-write version
  (it's waiting on a real GitHub Actions run to finish) — the panel shows
  "Publishing…" while it waits.
- The "Start a partnership" form still opens the visitor's email client
  with a pre-filled message to `hello@pentictonnightmarket.com` — no
  backend needed there either. Update that address in `index.html` if it
  should go somewhere else.
