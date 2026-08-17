# Self-Hosted Minecraft Server on an Old PC

A practical, start-to-finish guide to turning an old desktop or laptop into a self-hosted Paper Minecraft server — written from an actual build-out, including the real problems that came up along the way, not just the happy path.

## What you'll end up with

- A headless Ubuntu Server box running Paper, fully autonomous — starts on boot, restarts itself if it crashes or the power blips, no monitor or keyboard needed day to day
- Managed entirely over SSH from your main computer
- Reachable by friends over the internet, with a stable domain name instead of a raw IP
- Nightly backups, a locked-down firewall, and a whitelist so it isn't just sitting open on the internet

## Who this is for

Written for anyone reasonably comfortable with a terminal and basic Linux use. You don't need to be an expert — a fair amount of this guide is exactly the kind of troubleshooting a first-timer runs into — but some patience typing commands and reading error messages will help.

## The guide

| Part | Covers |
|---|---|
| [1 — Planning & Prerequisites](docs/01-planning-and-prerequisites.md) | Hardware reality check, software choices, what you'll need before you start |
| [2 — Installing Ubuntu Server](docs/02-installing-ubuntu-server.md) | Bootable USB, BIOS/UEFI settings, the OS install itself, first SSH login |
| [3 — Setting Up Paper](docs/03-setting-up-paper.md) | Java, a dedicated service account, downloading Paper, JVM tuning, first run |
| [4 — Making It Autonomous](docs/04-automation-and-hardening.md) | systemd, firewall, a fixed local IP, and BIOS gotchas that catch people out |
| [5 — Going Public](docs/05-going-public.md) | Port forwarding, checking for CGNAT, Dynamic DNS, whitelist, testing the connection |
| [6 — Maintenance & Troubleshooting](docs/06-maintenance-and-troubleshooting.md) | Backups, updates, a full command reference, and real troubleshooting scenarios |

## Example hardware used in this guide

Not a requirement — just what this particular build used, so the RAM/CPU numbers throughout have a concrete reference point:

- CPU: AMD Ryzen 5 3400G (4 cores / 8 threads)
- RAM: 8GB
- A wired ethernet connection (recommended over WiFi for a server)

Any similarly-aged mid-range PC should handle a 6–10 player survival server comparably.
