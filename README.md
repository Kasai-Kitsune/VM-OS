## VMOS 4.7 Virtual Machine Shell

A single-file, single-page fake retro OS. Boots like a late-90s VM-BIOS, drops you into a green-phosphor CRT desktop with a real window manager, and lets you poke around a simulated filesystem, terminal, and a full suite of built-in apps, all running against a resource governor that enforces a made-up 33 MHz / 32M spec as a real, breakable budget. One app makes genuine live web requests.

No build step, no dependencies, no backend. Open the HTML file in a browser and it runs.

The fixed disk's actuator chatter is emulated too, driven off real seek events on the virtual drive rather than a canned loop. If the clicking gets old, click the HD indicator in the taskbar tray (bottom right) to mute it; the amber/red LED keeps reporting drive activity regardless of whether the sound is on.

---

## What it actually is

This is a UI toy, not a real OS and not a real VM. Everything framed as "hardware," "BIOS," "kernel," or "host bridge" is flavor text rendered in HTML/CSS/JS. There is no sandboxing, no virtualization, no privilege model; it's a div wearing a trenchcoat.

That said, a few things in here are real:

* **The resource governor** is not decorative. RAM is a page-based, first-fit contiguous allocator over a 640K + 32,768K pool, so it genuinely fragments: a request can fail with plenty of total free memory if no single hole is big enough. CPU is modeled as a fixed 33 MHz core with no frequency scaling, so demand from open windows and background load actually slows animations, drags, and BASIC execution in real time, and pushes the machine toward a genuine (simulated) hang past ~150% load.
* **The fixed disk is a real, slow platter, not a prop.** A virtual IDE drive spins up on first cold boot (RPM ramp, FAT16 low-level format), and every file read/write is a seek against actual head position, not an instant lookup. A buffer cache (FCACHE) sits between the filesystem and the platters exactly like a real OS's page cache: freshly-opened files pay a cold seek, staying resident afterward until RAM pressure evicts the least-recently-used clean pages back to cold. DEFRAG operates on this same allocator and visibly closes real gaps, not a fake progress bar.
* **NETLINK** makes actual `fetch()` calls to the Wikipedia REST API and DuckDuckGo's Instant Answer API. Search results are genuine, not scripted.
* **Download interception:** any real `<a download>` link on the page gets caught before the browser's download manager sees it, fetched, and written into the simulated `C:\DOWNLOADS` folder instead. Nothing actually lands on your disk from inside the shell.
* Everything else—the filesystem, the terminal, the "kernel subsystems" in the boot log—is in-memory state that resets on reload.

---

## Features

* **Boot sequence:** fake POST/BIOS log with staged delays and beep SFX, skippable on keypress
* **Window manager:** draggable/resizable/maximizable windows, taskbar, start menu, right-click context menus, focus/z-order handling, `Alt+Tab` (real overlay, cycles on repeated Tab, commits on key-up), `Alt+F4` to close, `Ctrl+\`` to cycle windows, `Ctrl+Alt+Del` opens TASKMGR
* **Taskbar tray:** live NET status, a clock (click to open CLOCK.VMXE), and an HD indicator whose LED tracks real drive activity (amber = busy, red flare = an actual seek) with a click-to-mute toggle for the actuator chatter sound
* **5 CRT themes:** green (default), amber, ice, mono, magenta, swappable live via CONFIG
* **Resource governor:** an enforced VM-486 @ 33 MHz / 640K+32,768K budget. Every open app reserves real emulated RAM and CPU cycles; oversubscribe the CPU and the whole desktop genuinely stutters, exhaust a contiguous block of RAM and new programs get refused with an "Insufficient Memory" fault, and TASKMGR shows live load, per-process memory, and lets you End Task to actually free the reserved block
* **GREEN SCREEN OF DEATH:** a fatal-fault panic screen with glitch/tear effects and escalating noise bursts, triggered by simulated critical failures
* **Simulated filesystem:** a `C:\` tree (`SYSTEM`, `PROGRAMS`, `MAIL`, `DOWNLOADS`, `TEMP`) held in a JS object, browsable via File Explorer with right-click new/rename/delete, a Recycle Bin with restore/empty, and simulated `.vmzip` archives that "extract" into the current folder
* **TERMINAL.VMXE (VMSH):** a real small shell — `%VAR%` expansion and `set`, pipes (`dir | find "VMXE"`), redirection (`>`/`>>`), batch scripts (`run script.vmbat`), tab completion, and a `mem` command reporting live CPU/RAM load straight from the governor
* **NOTEPAD.VMXE:** text viewer/editor for simulated files, with Save/Save As/Export to Downloads
* **PAINT.VMXE:** simulated canvas, left-click draw / right-click erase
* **BASIC.VMXE:** a working VMBASIC interpreter
* **SOLITAIRE.VMXE:** two full games — Klondike and Pyramid (real pairs-sum-to-13 rules, coverage-aware)
* **MINES.VMXE:** a complete working Minesweeper clone
* **PLAYER.VMXE:** simulated media player
* **CALC.VMXE / CLOCK.VMXE / CHARMAP.VMXE:** calculator, clock, and character map utilities
* **FIND.VMXE:** find files by name across the simulated filesystem
* **DEFRAG.VMXE:** disk optimizer that visibly reworks the page allocator's fragmentation
* **CONFIG.VMXE:** theme, phosphor, and pixel-size tuning, plus reboot/saver shortcuts
* **SYSINFO / HELP.VMXE:** About panel and an in-universe help browser (topics include Memory & CPU, Terminal, Solitaire, About)
* **NETLINK.VMXE:** the one app that leaves the sandbox — live Wikipedia summary lookups and DuckDuckGo instant answers, rendered in retro-browser chrome, exempt from the CPU budget since the "lifting" happens on the far side of the bridge
* **Idle screensaver:** Matrix-style glyph rain after 3 minutes of inactivity, itself a real CPU load that throttles with the rest of the system
* **Warm reboot/shutdown:** both are in-page, not a host page reload. Reboot tears down windows and re-runs a shortened boot log while keeping the mounted disk and everything on it; shutdown parks the machine behind a power-off screen with a POWER ON button that resumes the same session. Neither wipes the simulated filesystem; only an actual browser refresh does that.

---

## Known limitations

* **No persistence:** refreshing the page wipes all simulated filesystem changes and resets the resource governor. This is by design, not a bug, but worth knowing before you write anything you care about into NOTEPAD (if one were to do that for some reason).
* **Font fallback:** The `VT323` font is referenced via `local('VT323')` with no bundled fallback. If you don't have it installed system-wide, it silently falls back to Courier New and the CRT look is noticeably less good.
* **Search bottleneck:** NETLINK's search quality is bottlenecked by DuckDuckGo's Instant Answer API, which returns rich results for encyclopedia-shaped queries and very little for anything else.
* **Governor curve is hand-tuned:** the CPU load-to-speed curve is a plausible-feeling three-piece function, not derived from an actual queueing model. It's tuned to feel right, not to be right.

---

## DISCLAIMER ⚠️

As I said, this is completely fake—it's a toy, a convincing fake/simulation. Any text displayed by it should not be taken seriously. I strongly discourage trying to use it for anything important.
