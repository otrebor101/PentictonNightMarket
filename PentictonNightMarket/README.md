# Penticton Night Market — site

A single-page site for the Penticton Night Market at Gyro Park. No backend required —
the whole thing runs as static files, and the "market updates" banner is powered by
this same GitHub repo.

## What's in here

```
index.html              the whole site (styles + markup + scripts in one file)
images/                  photos used in the hero and gallery sections
data/market-updates.json the live "market updates" banner content — starts as []
README.md                this file
```

## 1. Upload this to GitHub

1. Create a **public** repository (it must be public so the updates banner can be
   read by visitors without a login).
2. Upload all of these files/folders, keeping the same structure — `index.html` and
   `data/` and `images/` should all sit at the repo root.

## 2. Turn on GitHub Pages (free hosting)

1. In the repo, go to **Settings → Pages**.
2. Under "Build and deployment", set **Source** to "Deploy from a branch".
3. Branch: `main`, folder: `/ (root)`. Save.
4. GitHub will give you a URL like `https://your-username.github.io/your-repo/`
   within a minute or two — that's your live site.

## 3. Point the admin panel at your repo

Open `index.html`, find the `CONFIG` block near the bottom (search for `GH_OWNER`),
and fill in:

```js
const GH_OWNER  = "your-github-username";
const GH_REPO   = "your-repo-name";
const GH_BRANCH = "main";
const GH_PATH   = "data/market-updates.json";
```

Re-upload `index.html` with those values filled in (or edit it directly on GitHub —
click the file, click the pencil icon, edit, commit).

## 4. Create your admin token

1. GitHub → click your avatar → **Settings** → **Developer settings** →
   **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
2. **Repository access:** select "Only select repositories" → choose this repo.
   (Don't give it access to your other repos.)
3. **Permissions:** under "Repository permissions," set **Contents** to
   **Read and write**. Leave everything else as "No access."
4. Set an expiration date (30/60/90 days, or a custom date) — this way even a
   forgotten or leaked token stops working on its own.
5. Click **Generate token** and copy it immediately — GitHub only shows it once.
   Save it somewhere like a password manager.

## 5. Post an update

1. Open your live site, scroll to the footer, click **Admin**.
2. Paste your token in and log in.
3. Type an update and hit **Publish** — it writes straight to
   `data/market-updates.json` as a real commit in this repo. You can see the
   history of every change under the repo's **Commits** tab, and revert any of
   them if needed.
4. Click **Clear all updates** to empty the banner (also a commit, also revertible).

Visitors don't need a token — the banner is just a public read of that JSON file, so
it updates for everyone as soon as you publish.

## Notes

- The token is only ever held in the browser's memory for that tab — it is never
  written into any file, cookie, or storage. You'll need to paste it in again each
  time you want to edit updates.
- If you ever think a token leaked, revoke it immediately from **Developer
  settings → Fine-grained tokens** and generate a new one. Because it's scoped to
  only this repo with only Contents read/write, the worst case is someone edits
  this repo's files — not your account or other repos.
- The "Start a partnership" form on the site opens the visitor's email client with
  a pre-filled message to `hello@pentictonnightmarket.com` — no backend needed
  there either. Update that address in `index.html` if it should go somewhere else.
