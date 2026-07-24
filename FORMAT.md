# Writeup Markdown Format

## Front Matter

Front matter is optional, but it drives the sidebar and Box Info panel. Use this order for every writeup:

```yaml
---
title: Confidential
summary: One or two sentences for the tree and RSS feed.
date: 2026-06-30
tags: [Forensics, PDF, QR]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/...
---
```

- `date` must be `YYYY-MM-DD`; it drives sort order and the Published field.
- `difficulty` should be `easy`, `medium`, or `hard`.
- `os` is optional. Use `Linux`, `Windows`, `Windows Server 2016`, or `N/A` where helpful.
- `url` becomes the `Open on ...` button.

## CTF Events

Group a single competition in a lowercase-hyphenated subfolder under its platform. Split it into category subfolders if you like, and name each challenge file with the platform prefix:

```
HTB/cyber-apocalypse-2026/Hardware/htb-<challenge-slug>.md
HTB/cyber-apocalypse-2026/ICS/htb-<challenge-slug>.md
HTB/cyber-apocalypse-2026/AI/htb-<challenge-slug>.md
```

The site turns that into **Hack The Box → Cyber Apocalypse 2026 → Hardware / ICS / AI → challenge** in the tree (the event folder is title-cased automatically, so keep it lowercase with hyphens, not `CyberApocalypse2026`; short all-caps category names like `ICS`/`AI` are left as-is). Copy the event's `_template.md` for each challenge.

There is no `category` field, so also put the challenge class (Web, Pwn, Crypto, Reversing, Forensics, Misc, Hardware, Blockchain, AI) as the third `tags` entry so it stays searchable. Set `date` to the event date and `difficulty` to `easy`, `medium`, or `hard`.

**Embargoed events:** while a CTF is still running, stage it under an underscore-prefixed folder (`HTB/_cyber-apocalypse-2026/…`) so nothing is public. When you are clear to publish, `git mv` the folder to drop the underscore and everything goes live at once. Files that keep an underscore (`_README.md`, `_template.md`) stay private through the rename.

## Hidden Drafts

Use `THM/hidden/` for TryHackMe writeups that are still being drafted.

The website ignores the `hidden` folder, so Markdown files under `THM/hidden/` will not appear on the public site, RSS feed, or sitemap. When a writeup is ready, move it into the correct public folder:

- `THM/BLUE/`
- `THM/RED/`
- `THM/PURPLE/`

More generally, any file or folder whose name starts with `_` or `.`, or a folder named `hidden`, is ignored everywhere. That is why the `_template.md` files stay private, and it lets you keep a draft under any platform (for example `HTB/cyber-apocalypse-2026/_draft-foo.md`) until it is ready.

## Spoilers

Wrap any flag, password, hash, or answer you do not want fully visible in double pipes:

```markdown
The flag is ||THM{example}||.
```

This works everywhere, including code fences and tables. Unmarked `PREFIX{...}` flags are auto-redacted as a safety net, but explicit `||...||` is preferred because it also catches passwords and hashes. Inside a table cell, escape a literal double-pipe as `\|\|`.

## Code Fences

Always tag code fences with a language for highlighting and the language pill:

````markdown
```bash
nmap -sV -sC -Pn target.thm
```

```powershell
Get-LocalUser | Select-Object Name, LastLogon
```
````

Preferred tags include `bash`, `powershell`, `python`, `sql`, `json`, `yaml`, `http`, `spl`, `kql`, and `asm`. Use `text` for plain command output.

## Callouts

Use a blockquote whose first line starts with a bold label:

```markdown
> **Warning:** Don't run this against a box you don't own.
```

Supported labels are `Note:`, `Tip:`, and `Warning:`.

## Standard Markdown

Everything else is standard GFM: headings, tables with alignment, nested or task lists, bold, italic, strike, inline code, links, and images.
