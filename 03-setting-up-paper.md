# Part 3 — Setting Up Paper

With Ubuntu installed and SSH working, this part gets an actual Minecraft server running — manually, for now, before automating it in Part 4.

## Step 1: Install Java

Check Paper's own download page for the exact Java version your target Minecraft version requires before installing — getting this wrong means reinstalling later.

```bash
sudo apt install openjdk-XX-jre -y   # replace XX with the required version
java -version
```

Use the plain `-jre` package, **not** the `-headless` variant — Paper's own documentation specifically warns that the headless build is missing dependencies it needs, even though "headless" sounds like the obviously correct choice for a server with no display.

## Step 2: Create a dedicated service account

Running the Minecraft process as its own unprivileged account, rather than your personal admin login, means a compromised or buggy plugin can't touch anything outside its own folder:

```bash
sudo useradd -r -m -d /opt/minecraft -s /usr/sbin/nologin minecraft
sudo mkdir -p /opt/minecraft/server /opt/minecraft/backups
sudo chown -R minecraft:minecraft /opt/minecraft
```

One side effect worth knowing: your own login won't be able to browse into `/opt/minecraft` afterward without `sudo` — that's the permissions working correctly, not a problem to fix.

## Step 3: Download Paper

**Simplest method:** on your other computer, open [papermc.io/downloads/paper](https://papermc.io/downloads/paper), pick your Minecraft version, right-click the download button for the latest build and copy the link, then on the server:

```bash
cd /opt/minecraft/server
sudo -u minecraft wget -O paper.jar "PASTE_LINK_HERE"
```

**Or, a small script that always grabs the current latest stable build** — handy for this first download and for updates later:

```bash
#!/usr/bin/env bash
set -e
MC_VERSION="XX.X"   # set to your target version
UA="yourname/1.0 (contact@example.com)"   # Paper's API rejects generic user-agents
cd /opt/minecraft/server
URL=$(curl -s -H "User-Agent: $UA" \
  "https://fill.papermc.io/v3/projects/paper/versions/${MC_VERSION}/builds" \
  | jq -r '[.[] | select(.channel=="STABLE")][0].downloads."server:default".url')
sudo -u minecraft wget -O paper.jar "$URL"
```

```bash
sudo apt install jq -y   # needed for the script above
```

Then accept the EULA (a one-time legal requirement to actually run the server):

```bash
sudo -u minecraft bash -c 'echo eula=true > eula.txt'
```

## Step 4: Check how much RAM you actually have to work with

Before picking a heap size:

```bash
free -h
```

**On integrated-graphics hardware in particular, don't assume this matches your physical RAM.** Some motherboards reserve a chunk of system memory for the iGPU's frame buffer by default — on one real build, this alone accounted for nearly 2GB "missing" from an 8GB kit. If your total here looks notably lower than what's physically installed, jump ahead to Part 4's BIOS section and fix that first — it directly changes what heap size is safe to set here.

As a rule of thumb with a full amount visible: leave room for the OS, Java's own off-heap memory (thread stacks, JIT cache, network buffers — none of which counts against the heap number but is still real usage), and some slack before swap kicks in. On an 8GB box, a 5GB heap is a reasonable starting point.

## Step 5: Startup script with tuned JVM flags

```bash
sudo -u minecraft nano /opt/minecraft/server/start.sh
```

```bash
#!/usr/bin/env bash
cd /opt/minecraft/server
java -Xms5G -Xmx5G \
  -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 \
  -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch \
  -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M \
  -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 \
  -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 \
  -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 \
  -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 \
  -jar paper.jar --nogui
```

This flag set (widely known as "Aikar's flags") tunes Java's G1 garbage collector specifically for Minecraft's memory access patterns, and remains the standard recommendation for heaps under roughly 16–32GB — past that range, Java's ZGC collector starts winning out, but it's not worth the switch on a typical homelab-sized box.

```bash
sudo chmod +x /opt/minecraft/server/start.sh
sudo chown minecraft:minecraft /opt/minecraft/server/start.sh
```

## Step 6: Run it manually — before automating anything

Worth doing once by hand so you can watch it boot and catch any errors clearly, before wrapping it in systemd in Part 4:

```bash
sudo -u minecraft /opt/minecraft/server/start.sh
```

First startup takes a minute or two (generating spawn chunks, creating config files). You're looking for:

```
Done (XX.XXXs)! For help, type "help"
```

That confirms a clean boot. Shut it down properly (this matters — it saves world data correctly, unlike closing the terminal or `Ctrl+C`):

```
stop
```

---

**Next: [Part 4 — Making It Autonomous](04-automation-and-hardening.md)**
