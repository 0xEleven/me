---
title: Welcome to XIHacks — My Cyber Lab Notes
date: 2025-03-10
tags: meta, introduction
---

# Welcome to XIHacks

This is my personal lab notebook. Every CTF writeup, every tool I build, every rabbit hole I go down — it lands here.

## Why I'm doing this

The best hackers I know document obsessively. Not for clout — for **clarity**. Writing forces you to understand what you actually did, not just what you think you did.

> "If you can't explain it simply, you don't understand it well enough."

## What to expect here

- **CTF Writeups** — HackTheBox, TryHackMe, and competition challenges
- **Tool builds** — custom scripts, exploits, automation
- **Red team notes** — AD attacks, evasion, pivoting
- **Cloud security** — AWS/Azure misconfigs, IAM abuse
- **Web AppSec** — OWASP deep dives, API hacking

## The stack

This blog is pure HTML + Markdown. No frameworks, no build steps. I export from Notion, drop the `.md` file in the repo, add one line to `manifest.json`, and push. That's it.

```bash
# My entire publishing workflow
cp ~/notion-export/my-post.md blog/posts/
# edit manifest.json
git add . && git commit -m "new post" && git push
```

## Find me

- HTB: [Profile](https://app.hackthebox.com)
- THM: [Profile](https://tryhackme.com)
- Twitter: [@0xeleven](https://twitter.com)
- Email: mosesateka@gmail.com

Stay dangerous. Stay ethical.

— **0xeleven**
