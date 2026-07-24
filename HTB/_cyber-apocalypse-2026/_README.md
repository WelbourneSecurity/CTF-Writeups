# Cyber Apocalypse 2026 (staging)

Private staging for HackTheBox Cyber Apocalypse CTF 2026 writeups. The whole
folder is hidden from the public site, RSS feed and sitemap while the event is
running, because the name starts with an underscore.

## Where writeups go

One file per challenge, in its category folder, named `htb-<challenge-slug>.md`:

```
Hardware/htb-faulty-uart.md
ICS/htb-modbus-mayhem.md
AI/htb-prompt-heist.md
```

Copy `_template.md` for each one and fill it in. Set the third `tags` entry to
the category (Hardware, ICS, AI, …) so it stays searchable, and wrap the flag in
`||double pipes||`.

## Publishing after the event

When the event has ended and you are clear to publish, rename the folder to drop
the underscore:

```bash
git mv HTB/_cyber-apocalypse-2026 HTB/cyber-apocalypse-2026
git commit -m "Publish Cyber Apocalypse 2026 writeups"
git push
```

Everything goes live at once. The site lists it as **Hack The Box → Cyber
Apocalypse 2026 → Hardware / ICS / AI → challenge**, with static pages at
`/writeups/htb/cyber-apocalypse-2026/<category>/<slug>/`. This `_README.md` and
`_template.md` keep their underscores, so they stay private even after the
rename.
