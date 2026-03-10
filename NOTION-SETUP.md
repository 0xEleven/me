# 🔄 Notion → xihacks.com Auto-Publish Setup

Write in Notion → mark as Published → your blog updates automatically. No Git, no terminal.

---

## How it works

```
Notion DB  →  GitHub Action (every 6h)  →  manifest.json + .md files  →  Cloudflare Pages  →  xihacks.com
```

A GitHub Action runs every 6 hours, fetches all "Published" pages from your Notion database, converts them to Markdown, and pushes the files to your repo. Cloudflare Pages detects the push and deploys in ~30 seconds.

---

## Step 1 — Create your Notion database

1. Open Notion → **New page** → choose **Table** (full page)
2. Name it `xihacks Blog`
3. Add these exact properties:

| Property name | Type         | Notes |
|--------------|--------------|-------|
| `Title`      | Title        | Auto-exists |
| `Status`     | Select       | Add options: `Draft`, `Published`, `Archived` |
| `Slug`       | Text         | URL slug e.g. `hackthebox-lame-writeup` (leave blank to auto-generate) |
| `Tags`       | Multi-select | e.g. `ctf`, `hackthebox`, `linux` |
| `Excerpt`    | Text         | One line shown in blog listing |
| `Date`       | Date         | Publish date |

---

## Step 2 — Get your Notion API token

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **+ New integration**
3. Name it `xihacks-sync`, select your workspace
4. Set capabilities: **Read content** only
5. Click **Submit** → copy the **Internal Integration Token** (starts with `secret_`)

6. Go back to your Notion database page
7. Click **⋯** (top right) → **Add connections** → select `xihacks-sync`

---

## Step 3 — Get your database ID

Open your Notion database in the browser. The URL looks like:
```
https://www.notion.so/yourname/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx?v=...
```
The 32-character string is your **database ID**.

---

## Step 4 — Add secrets to GitHub

1. Go to your repo → **Settings** → **Secrets and variables** → **Actions**
2. Add two secrets:

| Secret name          | Value |
|---------------------|-------|
| `NOTION_TOKEN`      | Your integration token (`secret_...`) |
| `NOTION_DATABASE_ID`| Your 32-char database ID |

---

## Step 5 — Add the workflow files to your repo

Copy these files into your repo (they are in this zip):

```
.github/
├── workflows/
│   └── notion-sync.yml     ← the GitHub Action
└── scripts/
    ├── package.json         ← Node dependencies
    └── sync.js              ← the sync script
```

Push them to `main`:
```bash
git add .github/
git commit -m "add notion sync workflow"
git push
```

---

## Step 6 — Test it

1. Go to your repo → **Actions** tab
2. Click **Notion → Blog Sync** → **Run workflow** → **Run workflow**
3. Watch the logs — you should see your posts being fetched

If it shows green ✅ — you're done. New posts will sync every 6 hours automatically.

---

## Writing a post

1. Open your Notion database
2. Create a new row → write your post in the page body
3. Set **Status** = `Published`
4. Set **Date**, **Tags**, **Excerpt**
5. Wait up to 6 hours — or trigger manually from GitHub Actions

**To publish immediately:** Go to Actions → Run workflow manually.

---

## Tips

- **Slug**: Leave it blank and the title gets auto-converted (`My HTB Post` → `my-htb-post`)
- **Images**: Notion-hosted images will be referenced by URL in the markdown — they stay on Notion's CDN. For posts you want images stored in your repo, use the local editor (`blog/admin.html`)
- **Drafts**: Keep `Status = Draft` and they won't sync
- **Deleting**: Changing Status to `Archived` in Notion does NOT delete the file from your repo — you'd need to delete it manually. This prevents accidental deletions.
- **Force resync**: Trigger the workflow with "Force full sync" checked to regenerate all posts even if unchanged

---

## Troubleshooting

| Error | Fix |
|-------|-----|
| `NOTION_TOKEN not set` | Check your GitHub secret name is exactly `NOTION_TOKEN` |
| `Could not fetch content` | Make sure the integration has access to the page (Add connections in Notion) |
| `0 published posts found` | Check your Status property is named `Status` and the option is exactly `Published` |
| Posts not updating | The script skips posts where the file is newer than Notion's last-edited time. Use "Force full sync" |
