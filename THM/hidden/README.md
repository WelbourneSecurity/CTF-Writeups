---
title: Hidden Writeups
summary: Draft-only workspace for TryHackMe writeups that should not publish to the website yet.
date: 2026-07-06
tags: [TryHackMe, Drafts, Workflow]
difficulty: easy
os: N/A
url: N/A
---

# Hidden Writeups

Use this folder for TryHackMe writeups that are still being drafted, checked, or waiting for final placement.

Files in this folder do **not** publish to the website. The site indexer ignores `hidden` folders by name.

## Workflow

1. Draft the room in `THM/hidden/`.
2. Keep using the normal front matter, spoiler wrappers, and tagged code fences.
3. When the writeup is ready, move it into one of the public folders:
   - `THM/BLUE/`
   - `THM/RED/`
   - `THM/PURPLE/`
4. Commit and push after moving it. The website will pick it up after the normal index rebuild.

## Naming

Use the final filename from the start so moving it is clean:

```text
thm-room-name.md
```

For example:

```text
THM/hidden/thm-example-room.md
THM/PURPLE/thm-example-room.md
```
