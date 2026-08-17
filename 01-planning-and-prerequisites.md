# Part 1 — Planning & Prerequisites

Before touching any hardware, it's worth deciding a few things up front — they'll save you from having to backtrack later.

## Hardware: what's actually needed

You don't need much. A machine that was mid-range in the last decade is generally plenty:

- **CPU:** Minecraft's main tick loop is mostly single-threaded, so per-core speed matters more than core count. A quad-core from the last ~8 years is comfortably enough for 6–10 players on Paper (this guide's example machine is a Ryzen 5 3400G, 4c/8t).
- **RAM:** 8GB is a reasonable floor for a small survival server (6–10 players) once you've reserved room for the OS alongside the game itself — the exact split is covered in Part 3. 4GB can work for a handful of players if you're tight on options, but 8GB gives you real breathing room.
- **Storage:** Doesn't need to be large. An SSD is a meaningfully better experience than a spinning HDD if you have the choice — faster chunk reads/writes, faster backups — but an HDD will still work for a small server.
- **Network:** A wired ethernet connection, not WiFi. More stable, one less variable to debug later.
- **Integrated graphics is fine.** You don't need a discrete GPU — the server runs headless (no desktop environment, no games being rendered on it). Worth knowing ahead of time: some motherboards reserve a chunk of system RAM for integrated graphics by default, which can eat into your budget unexpectedly. Part 4 covers how to check for and fix this.

## Software choices

**Operating system: Ubuntu Server LTS.** Headless by default (no desktop environment to strip out), long support window, and if you're already comfortable with Linux basics, no real learning curve. This guide uses the current LTS release throughout — substitute whichever is current when you're following along.

**Server software: Paper.** Vanilla-compatible (same gameplay, same worlds, same experience for players) but with substantial performance optimizations and support for plugins if you ever want them. There's rarely a reason to run vanilla's own server jar over Paper for a self-hosted setup — you get the performance and plugin option for free, with no gameplay downside if you never install a single plugin.

**A note on Minecraft's versioning:** Minecraft moved to a `year.drop` versioning scheme in 2026 (`26.1`, `26.2`, etc.) instead of the old `1.x` numbering. Whatever the current scheme looks like when you're reading this, two things are worth checking before you commit to a version:

1. **Which Java version does it need?** This is usually clearly stated on Paper's own download page — check it before installing Java in Part 3, since guessing wrong means reinstalling later.
2. **Have the plugins you care about updated for it yet?** Major version jumps (like the 2026 scheme change) tend to leave the plugin ecosystem playing catch-up for a while. If there's a specific plugin you're building around, check its current compatibility before committing to the newest Minecraft version — running one version branch behind is a completely reasonable trade-off for better plugin support.

## What you'll need before starting

- The old PC itself, plus a keyboard and monitor **temporarily** (only needed for the initial install — everything after that happens over SSH).
- A second computer (laptop or otherwise) to create the bootable USB installer from.
- A blank USB drive (4GB+ is plenty).
- Physical access to your router's admin page (you'll need this twice: once for a fixed local IP, once for port forwarding later).

## A short pre-flight checklist

A few things are worth checking **before** you start installing anything — cheap to verify now, annoying to discover halfway through:

- **Do you know your router's admin login?** If you've never changed it from default, it's often printed on a sticker on the router itself.
- **Does your motherboard support UEFI boot?** Virtually anything from the last ~10–12 years does, but worth a quick check if your hardware is genuinely old.
- **Are you prepared for this to wipe the drive?** This guide assumes the old PC is being dedicated entirely to this — back up anything on it first if it isn't already empty/spare hardware.

One thing that's *not* worth stressing about in advance: whether your home internet can support letting friends connect from outside. That's worth checking, but it's a Part 5 concern — nothing about the OS/server setup depends on it, so there's no reason to hold up starting.

---

**Next: [Part 2 — Installing Ubuntu Server](02-installing-ubuntu-server.md)**
