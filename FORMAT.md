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
