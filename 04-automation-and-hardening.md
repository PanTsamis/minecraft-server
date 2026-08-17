# Part 4 — Making It Autonomous

At this point the server runs — but only while you're manually watching it, and only until the next reboot. This part covers turning that into something that starts itself, stays running, survives a power cut, and isn't wide open on the network.

## Step 1: systemd service (the actual "autonomous" part)

Install `screen` first — it lets you attach to the server's live console later without having been the one who started it:

```bash
sudo apt install screen -y
```

Create the service definition:

```bash
sudo nano /etc/systemd/system/minecraft.service
```

```ini
[Unit]
Description=Minecraft Paper Server
After=network.target

[Service]
User=minecraft
WorkingDirectory=/opt/minecraft/server
ExecStart=/usr/bin/screen -DmS mcserver /opt/minecraft/server/start.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Why `Restart=always` and not the seemingly-more-correct `Restart=on-failure`:** `on-failure` only restarts the process on a crash (non-zero exit code). A *clean* shutdown — for instance, someone accidentally typing `stop` in the console instead of using systemd to stop it — counts as success and won't trigger a restart, silently leaving the server down until someone notices. `always` restarts it regardless of why it stopped, which is what "autonomous" actually requires in practice.

**The trade-off:** with this setting, typing `stop` in the live console no longer takes the server down for good — systemd brings it back within ~10 seconds. For genuine maintenance shutdowns, stop it from a normal SSH prompt instead, which systemd treats as intentional and won't override:

```bash
sudo systemctl stop minecraft
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now minecraft.service
sudo systemctl status minecraft.service
```

Should show `active (running)`. Day-to-day console access from here on:

```bash
sudo -u minecraft screen -r mcserver   # attach — see live logs, type commands
# Ctrl+A then D to detach without stopping it — never just close the terminal
```

## Step 2: Give it a fixed local IP

A DHCP-assigned IP can change on you, which is annoying once you've started referencing it in scripts and bookmarks. Reserve it permanently via your router instead of configuring it statically on the PC — this survives OS reinstalls and is generally more reliable:

1. On the server, get its MAC address: `ip a` — look for the `link/ether` line under your ethernet interface.
2. Find your router's admin address: `ip r` — the IP shown after "default via."
3. Log into that address from a browser on your other computer.
4. Find the DHCP/address reservation section — naming varies a lot by router brand (commonly "DHCP Reservation," "Address Reservation," "Static Leases," or "IP-MAC Binding").
5. Bind the server's MAC address to an IP.

**Note from experience:** some router firmware won't let you reserve the IP the server is *currently* leasing — only unused addresses show as selectable. If that happens, just pick a different free address; the server's IP will change to that new one going forward, so use it in every command from this point on rather than the original.

Verify it stuck:

```bash
sudo reboot
# wait ~30s, reconnect with the reserved IP, then:
ip a
```

## Step 3: Firewall

```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 25565/tcp
sudo ufw enable
sudo ufw status
```

This should show SSH and Minecraft's port allowed, everything else implicitly denied.

## Step 4: BIOS — power resilience and reclaiming lost RAM

Two or three settings worth fixing in the same BIOS trip.

**Survive power outages.** Find **"AC Power Recovery"**, **"Restore on AC/Power Loss,"** or similar (naming and location vary by motherboard brand — commonly under an Advanced or Power-related tab). Set it to **Power On** (avoid "Last State" — if the power cut while it happened to be off, it'd stay off). Now if power blips, the PC turns itself back on, boots to the login prompt, and systemd brings the server back up automatically — no human required.

**If Part 3's `free -h` check showed less RAM than physically installed**, this is very likely why: on integrated-graphics boards, look under a **Chipset → GFX/Graphics Configuration** style menu for **"UMA Frame Buffer Size."** If it's set to **Auto**, the board may be reserving a surprisingly large chunk for the iGPU's frame buffer — memory a headless server doesn't need. Set it to the smallest available option (commonly **64M**). Reboot and re-check `free -h` — total should jump up close to the full physical amount.

**Optional, software not BIOS:** Ubuntu Server reserves some RAM (~512MB) for a crash-dump kernel via the `kdump-tools` package — genuinely useful on a production box someone gets paged for, less so on a homelab server:

```bash
sudo apt purge kdump-tools -y
sudo update-grub
sudo reboot
```

**If either RAM fix changed your total meaningfully**, go back to Part 3 and reconsider whether the JVM heap size should be adjusted to match.

---

**Next: [Part 5 — Going Public](05-going-public.md)**
