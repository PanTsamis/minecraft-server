# Part 2 — Installing Ubuntu Server

This part covers everything from an empty USB stick to a fresh Ubuntu Server install you can reach over SSH.

## Step 1: Create the bootable installer

On your second computer:

1. Download the current Ubuntu Server LTS ISO from [ubuntu.com/download/server](https://ubuntu.com/download/server).
2. Flash it to a USB stick:
   - **Windows:** [Rufus](https://rufus.ie)
   - **Any OS:** [balenaEtcher](https://etcher.balena.io)
   - **Linux/macOS terminal:**
     ```bash
     # Double-check the device name first with lsblk / diskutil list — this is destructive
     sudo dd if=ubuntu-XX.XX-live-server-amd64.iso of=/dev/sdX bs=4M status=progress conv=fsync
     ```

### If you're using Rufus specifically

Once you select the ISO, Rufus will likely ask whether to write in **"ISO Image mode"** or **"DD Image mode"** — choose **ISO Image mode**. For partition scheme and target system, **GPT + UEFI (non-CSM)** is the right choice for essentially any PC built in the last decade or so — it's the modern standard with no real downside over legacy BIOS/MBR unless your hardware genuinely predates UEFI support. File system can stay at the default (FAT32).

Click **Start**, confirm any prompt about downloading extra UEFI boot files (say yes), and confirm the warning that the USB drive will be erased.

## Step 2: Boot the target PC from the USB

1. Plug the USB into the old PC, power it on, and hit the boot-menu key — commonly `F11`, `F12`, `Del`, or `Esc`, varies by motherboard brand.
2. In the boot menu, select the USB drive. **If you see two entries for it** — one plain and one prefixed `UEFI:` — pick the `UEFI:` one.
3. Select **"Try or Install Ubuntu Server"** and press Enter.

A black screen for anywhere from a few seconds to a couple of minutes right after the initial boot messages is normal — that's the switch from the kernel's boot console to the actual installer interface, not a freeze. Give it time before assuming something's wrong.

## Step 3: Walk through the installer

- **Language / keyboard layout:** whatever fits you.
- **Base install — "Ubuntu Server" vs "Ubuntu Server (minimized)":** minimized is a good fit for a single-purpose box like this — it skips a bunch of packages a general-purpose server ships with but a dedicated Minecraft box will never use. **Trade-off worth knowing upfront:** it also strips out some everyday tools — `nano` (text editor), `cron` (scheduled tasks), `dnsutils` (gives you the `dig` command) — that you'll need to manually `sudo apt install` the first time each is needed later in this guide. Regular "Ubuntu Server" skips this trade-off entirely if you'd rather have them available from the start.
- **Skip "search for third-party drivers"** — mainly for WiFi chip firmware or exotic RAID controllers, not relevant on a wired connection.
- **Network:** if using wired ethernet, make sure the cable is plugged in *before* you reach this screen — the installer probes for a link around this point, and a cable plugged in afterward may show "autoconfiguration failed" until you either select the interface → Edit IPv4 → Save (to force a retry) or back out and step into the screen again. DHCP is fine for now; you'll pin a permanent IP from your router in Part 4.
- **Proxy / mirror:** leave blank / accept the default. You'll see a brief "Reading package lists" step afterward — that's normal `apt` output as it syncs against the mirror, not a separate stuck process.
- **Storage:** guided, **use entire disk**. Leave **LVM unchecked** (unnecessary abstraction for a single-disk single-purpose box) — this also means the **LUKS encryption** option won't apply, which is what you want: encryption requires a password typed at every boot, which breaks the "restart itself unattended after a power blip" goal from Part 4. When asked, allow a **swap file** (2–4GB) as an out-of-memory safety net, not something to actually rely on day-to-day.
- **Profile setup:** "Your name" is cosmetic. "Server's name" is the hostname — anything memorable works. **"Username" is your own personal login** — the account you'll SSH in as, with sudo rights — not a separate account for the Minecraft process itself (that gets created separately in Part 3, specifically so the game process doesn't run with admin rights).
- **Ubuntu Pro (optional):** free for personal use up to 5 machines, extends security-patch coverage and adds kernel live-patching (updates without needing a reboot). Requires signing in with an Ubuntu One account to get a token. Entirely skippable — you can always run `sudo pro attach <token>` after the fact if you change your mind.
- **Install OpenSSH server: yes.** This is what lets you manage the box remotely afterward instead of needing a monitor/keyboard permanently attached. Leave **password authentication enabled** for now (you don't have an SSH key set up yet) — locking this down to key-only auth later is a reasonable follow-up once everything's confirmed working.
- **Featured server snaps:** skip all of them (Kubernetes, Nextcloud, monitoring stacks, cloud CLI tools, etc.) — none apply here, and everything this guide actually needs gets installed explicitly via `apt` later regardless.

## Step 4: Finish and reboot

Once installation completes, you'll get a choice between **"View full log"** and **"Reboot now."** Pick **Reboot now** — it'll prompt you to remove the USB stick and press Enter to continue.

It boots to a plain text login prompt. That's expected and correct for a headless server — not a bug, not a missing desktop.

## Step 5: First login and SSH access

At the server itself (one-time, using the keyboard/monitor):

```bash
ip a
```

Find your ethernet interface in the output and note the IP address next to it (something like `192.168.1.X`).

From your other computer:

```bash
ssh yourusername@192.168.1.X
```

The first connection will show a host key fingerprint and ask you to confirm — type `yes` (the full word) and press Enter. This is a one-time trust check per server, not a bug. Then enter your password (no characters or dots will show as you type — that's normal terminal behavior, not a stuck input).

Once you're in, update the system:

```bash
sudo apt update && sudo apt full-upgrade -y
```

If it indicates a new kernel was installed (often a `*** System restart required ***` banner), reboot and reconnect:

```bash
sudo reboot
```

Wait about 30 seconds, then SSH back in with the same command as before. From this point forward, you can disconnect the monitor and keyboard from the server entirely — everything else in this guide happens over SSH.

---

**Next: [Part 3 — Setting Up Paper](03-setting-up-paper.md)**
