## VMOS 4.7 — Virtual Machine Shell

A single-file, single-page fake retro OS. Boots like a late-90s VM-BIOS, drops you into a green-phosphor CRT desktop with a real window manager, and lets you poke around a simulated filesystem, terminal, text editor, image viewer, and a couple of built-in apps, including one that makes genuine live web requests.

No build step, no dependencies, no backend. Open the HTML file in a browser and it runs.

---

## What it actually is

This is a UI toy, not a real OS and not a real VM. Everything framed as "hardware," "BIOS," "kernel," or "host bridge" is flavor text rendered in HTML/CSS/JS. There is no sandboxing, no virtualization, no privilege model—it's a div wearing a trenchcoat.

That said, a couple of things in here are real:

* **NETLINK** makes actual `fetch()` calls to the Wikipedia REST API and DuckDuckGo's Instant Answer API. Search results are genuine, not scripted.
* **Download interception:** any real `<a download>` link on the page gets caught before the browser's download manager sees it, fetched, and written into the simulated `C:\DOWNLOADS` folder instead. Nothing actually lands on your disk from inside the shell.
* Everything else—the filesystem, the terminal, the "kernel subsystems" in the boot log—is in-memory state that resets on reload.

---

## Features

* **Boot sequence:** fake POST/BIOS log with staged delays and beep SFX, skippable on keypress
* **Window manager:** draggable/resizable windows, taskbar, start menu, focus/z-order handling, `Ctrl+\`` to cycle windows
* **4 CRT themes:** green (default), amber, ice, mono, magenta, swappable live via CONFIG
* **Simulated filesystem:** a `C:\` tree (`SYSTEM`, `MAIL`, `DOWNLOADS`, `TEMP`) with a File Explorer supporting right-click new/rename/delete, all held in a JS object, not real disk I/O
* **TERMINAL.VMXE:** a small shell with a help command
* **PAD:** plain text viewer/editor for `.txt`/`.vmtxt` files in the simulated FS
* **Image viewer:** checkerboard-backed viewer for simulated `.png`/`.jpg` entries
* **NETLINK.VMXE:** the one app that leaves the sandbox: live Wikipedia summary lookups and DuckDuckGo instant answers, rendered in retro-browser chrome
* **MINES.VMXE:** a complete working Minesweeper clone
* **CONFIG.VMXE:** theme, phosphor, and pixel-size tuning
* **Idle screensaver:** Matrix-style glyph rain after 3 minutes of inactivity
* **Fake reboot/shutdown:** animated transitions, reboot just reloads the page, shutdown blanks it with a "safe to close this tab" message

---

## Known limitations

* **No persistence:** refreshing the page wipes all simulated filesystem changes. This is by design, not a bug, but worth knowing before you write anything you care about into PAD (if one were to do that for some reason).
* **Font fallback:** The `VT323` font is referenced via `local('VT323')` with no bundled fallback. If you don't have it installed system-wide, it silently falls back to Courier New and the CRT look is noticeably less good.
* **Search bottleneck:** NETLINK's search quality is bottlenecked by DuckDuckGo's Instant Answer API, which returns rich results for encyclopedia-shaped queries and very little for anything else.
* **Single file complexity:** It's a single ~2,200-line HTML file. Great for portability, bad for anyone trying to extend it—splitting into modules would be the first real refactor if this grows further.
* **Query state bug:** Each search query can only be used once for some reason, haven't found the problem yet. Resets on refresh.

---

## DISCLAIMER ⚠️

As I said, this is completely fake—it's a toy, a convincing fake/simulation. Any text displayed by it should not be taken seriously. I strongly discourage trying to use it for anything important.
