# Part 6 — Maintenance & Troubleshooting

The server's live, autonomous, and reachable. This last part covers keeping it that way: backups, updates, a command reference for ongoing admin work, and fixes for the real problems that tend to come up.

## Backups

```bash
sudo apt install cron -y   # skip if already installed
sudo crontab -u minecraft -e
```

Add a nightly backup with a week of rolling history:

```cron
0 4 * * * tar -czf /opt/minecraft/backups/world-$(date +\%F).tar.gz -C /opt/minecraft/server world world_nether world_the_end
30 4 * * * find /opt/minecraft/backups -name "world-*.tar.gz" -mtime +7 -delete
```

Confirm it saved:

```bash
sudo crontab -u minecraft -l
```

**Restoring a backup** — the command you hope never to need, but should know before you do:

```bash
sudo systemctl stop minecraft
sudo -u minecraft tar -xzf /opt/minecraft/backups/world-2026-08-20.tar.gz -C /opt/minecraft/server
sudo systemctl start minecraft
```

(swap in the real filename — check with `ls -lh /opt/minecraft/backups/` first). This overwrites the current world files, so double-check the filename before running it.

## Updating Paper

Re-run the download step from Part 3 (either the manual browser method or the scripted one) to grab the current latest build, then:

```bash
sudo systemctl restart minecraft
```

The new jar doesn't take effect until the process restarts.

## Command reference

### Service control

| Command | What it does |
|---|---|
| `sudo systemctl start minecraft` | Starts the server if stopped. |
| `sudo systemctl stop minecraft` | The real way to stop it — saves cleanly and won't auto-restart, unlike typing `stop` in the console (see Part 4). |
| `sudo systemctl restart minecraft` | Needed after editing `start.sh`, `server.properties`, or the service file. |
| `sudo systemctl status minecraft` | Is it running, for how long, current memory use. |
| `sudo systemctl daemon-reload` | Run after editing `minecraft.service` itself, before restarting. |

### Console & logs

| Command | What it does |
|---|---|
| `sudo -u minecraft screen -r mcserver` | Attach to the live console. |
| `Ctrl+A` then `D` | Detach without stopping the server — always use this to leave. |
| `journalctl -u minecraft -f` | Watch the log without attaching (read-only, `Ctrl+C` to stop). |

### In-game commands (typed in the live console)

| Command | What it does |
|---|---|
| `list` | Who's online. |
| `tps` | Server performance over 1/5/15 min — should read close to 20.0. |
| `whitelist add / remove / list` | Manage who can join. |
| `op` / `deop` | Grant/revoke admin powers — only for people you fully trust. |
| `kick` / `ban` | Remove a player, temporarily or permanently. |
| `save-all` | Force an immediate world save — good before anything risky. |

### Health checks

| Command | What it does |
|---|---|
| `free -h` | RAM/swap usage — swap should stay near 0 in normal play. |
| `df -h` | Disk space (backups accumulate over time). |
| `curl ifconfig.me` | Current public IP. |
| `dig +short yourname.duckdns.org` | Confirms DuckDNS is resolving correctly. |
| `sudo ufw status` | Confirms firewall rules are still correct. |

## Troubleshooting

**RAM shown by `free -h` is noticeably less than what's physically installed.**
Common on integrated-graphics motherboards — the BIOS may be reserving a large chunk for the iGPU's frame buffer by default. Covered in Part 4: look for "UMA Frame Buffer Size" and set it to a small fixed value instead of Auto.

**The server goes down and doesn't come back on its own.**
If `Restart=on-failure` is set instead of `Restart=always` in the service file, a *clean* exit (e.g. someone typing `stop` in the console) won't trigger an automatic restart — only crashes do. Switch to `Restart=always` (Part 4) so it comes back regardless of why it stopped.

**A command says "command not found" — `nano`, `crontab`, `dig`, etc.**
If you chose the minimized Ubuntu Server install in Part 2, several everyday tools aren't included by default. Install the missing one directly: `sudo apt install nano -y` (or `cron`, or `dnsutils` for `dig`).

**"Permission denied" browsing into `/opt/minecraft`.**
Expected — the dedicated service account's home directory is locked down from other users by design (Part 3). Use `sudo` for anything that needs to read inside it.

**A remote friend lags badly despite having "good internet" themselves.**
Their internet speed isn't actually the relevant number — **your home connection's upload bandwidth** is, since that's what every byte the server sends to them has to travel through. This is a common bottleneck on connections with asymmetric speeds (fast download, slow upload), like most ADSL and some cable plans. To diagnose:
1. Check server performance first: `tps` in the console. If it's near 20.0, the server itself is fine and this is network-related.
2. Check whether anything else in your house is uploading heavily at the same time (cloud backups, video calls) — it shares the same limited pipe.
3. Have them check their actual ping via the F3 debug screen in-game — a real number is far more useful than "it lags."
4. Try trimming `view-distance`/`simulation-distance` further (Part 5) — less data sent per tick eases the load on a constrained upload connection.
5. If it's consistently bad regardless, this may simply be a ceiling of your connection type — a faster plan (particularly one with better upload, like fiber) is the actual fix, not further server tuning.

**Friends still can't connect after port forwarding is set up.**
Re-check the CGNAT test from Part 5 first (it's the most common root cause) — then confirm the server is actually running, the port forward rule saved with the right internal IP, and their username is genuinely on the whitelist.

## Where to go from here

The server as built is deliberately vanilla-feeling — no plugins required for any of this guide. If you want to extend it, Paper supports plugins for things like grief-protection/rollback logging, quality-of-life commands (teleport requests, spawn warps), Discord chat bridging, and live web maps of the world. Minecraft's plugin ecosystem moves fast and version compatibility shifts constantly, so whatever you're considering, check its current compatibility with your exact Minecraft version on its download page before installing — don't assume a plugin that worked on an older version already supports whatever you're running now.

---

**[Back to the guide index](README.md)**
