## VMOS 9.7 Virtual Machine Shell

A single-file, single-page fake retro OS. Boots like a late-90s VM-BIOS, drops you into a green-phosphor CRT desktop with a real window manager, and lets you poke around a simulated filesystem, terminal, and a full suite of built-in apps, all running on a real emulated 32-bit CPU against a resource governor that enforces a 33 MHz / 32M spec as a real, breakable budget. One app makes genuine live web requests.

No build step, no dependencies, no backend. Open the HTML file in a browser and it runs.

---

## What it actually is

This is a UI toy, not a real OS and not a real VM. Everything framed as "hardware," "BIOS," "kernel," or "host bridge" is flavor text rendered in HTML/CSS/JS. There is no sandboxing, no virtualization, no privilege model; it's a div wearing a trenchcoat.

That said, a few things in here are real:

* **The CPU is a real, cycle-accurate processor.** Under the hood is the Vulp-33, a from-scratch 32-bit CISC core (486-class, 33 MHz) with its own instruction set, registers, on-die cache, interrupt table, and two-pass assembler. It doesn't fake being slow; it fetches, decodes, and executes real machine code and counts real clock cycles per instruction. It's wired in like a physical chip: from power-on it never stops, idling in a firmware `HLT` loop the timer IRQ wakes, and opening a program posts a real job to it that spins up in cycle-accurate time, so a bigger app genuinely takes longer to launch. Time is honest: one simulated second is one real second at 33 MHz.
* **The fixed disk is a real, slow platter, not a prop.** A virtual IDE drive spins up on first cold boot (RPM ramp, FAT16 low-level format), and every file read/write is a seek against actual head position, not an instant lookup. A buffer cache (FCACHE) sits between the filesystem and the platters exactly like a real OS's page cache: freshly-opened files pay a cold seek, staying resident afterward until RAM pressure evicts the least-recently-used clean pages back to cold. DEFRAG operates on this same allocator and visibly closes real gaps, not a fake progress bar.
* **The resource governor** is not decorative. RAM is a page-based, first-fit contiguous allocator over a 640K + 32,768K pool, so it genuinely fragments: a request can fail with plenty of total free memory if no single hole is big enough. Load from open windows and background work actually slows animations, drags, and BASIC execution in real time, and pushes the machine toward a genuine (simulated) hang past ~150% load. The disk cache draws from this same pool, so apps and files really do compete for memory.
* **NETLINK** makes actual `fetch()` calls to the Wikipedia REST API and DuckDuckGo's Instant Answer API. Search results are genuine, not scripted.
* **Download interception:** any real `<a download>` link on the page gets caught before the browser's download manager sees it, fetched, and written into the simulated `C:\DOWNLOADS` folder instead. Nothing actually lands on your disk from inside the shell.
* Everything else—the terminal, the "kernel subsystems" in the boot log—is in-memory state that resets on reload.

---

## Features

* **Boot sequence:** fake POST/BIOS log with staged delays and beep SFX, skippable on keypress; a cold boot really powers on the Vulp-33 (with a POST self-test) and spins up + formats the IDE disk before handing off
* **Window manager:** draggable/resizable/maximizable windows, taskbar, start menu, right-click context menus, focus/z-order handling, `Alt+Tab` (real overlay, cycles on repeated Tab, commits on key-up), `Alt+F4` to close, `Ctrl+\`` to cycle windows, `Ctrl+Alt+Del` opens TASKMGR; titlebar drag/resize use pointer events, so windows drag on touchscreens too (touch zone limited to the titlebar/resize handle, so scrolling inside a window still works)
* **Taskbar tray:** live NET status, a clock (click to open CLOCK.VMXE), and an HD indicator whose LED tracks real drive activity (amber = busy, red flare = an actual seek); the drive is silent by design
* **5 CRT themes:** green (default), amber, ice, mono, magenta, swappable live via CONFIG
* **Resource governor:** an enforced Vulp-33 @ 33 MHz / 640K+32,768K budget. Every open app reserves real emulated RAM and CPU cycles; oversubscribe the CPU and the whole desktop genuinely stutters, exhaust a contiguous block of RAM and new programs get refused with an "Insufficient Memory" fault, and TASKMGR shows live load, per-process memory, and lets you End Task to actually free the reserved block
* **GREEN SCREEN OF DEATH:** a fatal-fault panic screen with glitch/tear effects and escalating noise bursts, triggered by simulated critical failures
* **Simulated filesystem:** a `C:\` tree (`SYSTEM`, `PROGRAMS`, `MAIL`, `DOWNLOADS`, `TEMP`) backed by the real FAT16 volume, browsable via File Explorer with right-click new/rename/delete, a Recycle Bin with restore/empty, and simulated `.vmzip` archives that "extract" into the current folder
* **TERMINAL.VMXE (VMSH):** a real small shell — `%VAR%` expansion and `set`, pipes (`dir | find "VMXE"`), redirection (`>`/`>>`), batch scripts (`run script.vmbat`), tab completion, and a `mem` command reporting live CPU/RAM load plus real Vulp-33 telemetry (cycles, CPI, cache hit-rate, uptime) straight from the chip
* **NOTEPAD.VMXE:** text viewer/editor for simulated files, with Save/Save As/Export to Downloads
* **PAINT.VMXE:** simulated canvas, left-click draw / right-click erase
* **BASIC.VMXE:** a working VMBASIC interpreter
* **SOLITAIRE.VMXE:** two full games — Klondike and Pyramid (real pairs-sum-to-13 rules, coverage-aware)
* **MINES.VMXE:** a complete working Minesweeper clone
* **PLAYER.VMXE:** simulated media player
* **CALC.VMXE / CLOCK.VMXE / CHARMAP.VMXE:** calculator, clock, and character map utilities
* **FIND.VMXE:** find files by name across the simulated filesystem; content search demand-reads each candidate off the disk, like a real grep -r
* **DEFRAG.VMXE:** disk optimizer that visibly reworks the page allocator's fragmentation
* **CONFIG.VMXE:** theme, phosphor, and pixel-size tuning, plus reboot/saver shortcuts
* **SYSINFO / HELP.VMXE:** About panel and an in-universe help browser (topics include Memory & CPU, Terminal, Solitaire, About)
* **NETLINK.VMXE:** the one app that leaves the sandbox — live Wikipedia summary lookups and DuckDuckGo instant answers, rendered in retro-browser chrome, exempt from the CPU budget since the "lifting" happens on the far side of the bridge
* **Idle screensaver:** Matrix-style glyph rain after 3 minutes of inactivity, itself a real CPU load that throttles with the rest of the system
* **Warm reboot/shutdown:** both are in-page, not a host page reload. Reboot tears down windows and re-runs a shortened boot log while keeping the mounted disk and everything on it; shutdown parks the machine behind a power-off screen with a POWER ON button that resumes the same session. Neither wipes the simulated filesystem; only an actual browser refresh does that.

---

## Known limitations

* **No persistence:** refreshing the page wipes all simulated filesystem changes and resets the CPU and resource governor. This is by design, not a bug, but worth knowing before you write anything you care about into NOTEPAD (if one were to do that for some reason).
* **The CPU runs its own chip, not the whole OS.** The Vulp-33 is genuinely cycle-accurate, but the VMOS shell is JavaScript and can't be recompiled into Vulp-33 machine code, so the chip really runs discrete workloads (POST, program spin-up, its assembler) in real time while the desktop UI is still hosted by the browser. Same honest hybrid as the disk.
* **Font fallback:** The `VT323` font is referenced via `local('VT323')` with no bundled fallback. If you don't have it installed system-wide, it silently falls back to Courier New and the CRT look is noticeably less good.
* **Search bottleneck:** NETLINK's search quality is bottlenecked by DuckDuckGo's Instant Answer API, which returns rich results for encyclopedia-shaped queries and very little for anything else.

---

## 📖 The two files in foundational files are emulators with an attached workbench that I had to build independently before I could integrate them into the main project. One is a fictional chip with a very precise specs albeit; and the other is a HDD.

---

## DISCLAIMER ⚠️

As I said, this is completely fake—it's a toy, a convincing fake/simulation. Any text displayed by it should not be taken seriously. I strongly discourage trying to use it for anything important.
