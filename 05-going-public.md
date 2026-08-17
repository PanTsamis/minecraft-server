# Part 5 — Going Public

Everything so far works entirely inside your home network. This part opens it up so friends can actually connect from outside.

## Step 1: Check for CGNAT before anything else

This determines whether port forwarding will work *at all* — worth confirming first so you don't spend time configuring something that can't succeed.

**What it is:** some ISPs put multiple customers behind a single shared public IP address (Carrier-Grade NAT). If that's your situation, no amount of router configuration will let inbound connections reach you — you don't actually control a real public IP to forward a port on.

**The correct test:** compare your router's own WAN IP (visible on its status/overview page) against what an external site reports your public IP as, checked *from a device on your home network*:

```bash
curl ifconfig.me
```

- **Match** → you have a genuine public IP, port forwarding will work normally.
- **Different** → you're behind CGNAT.

**A test that does *not* work, despite seeming intuitive:** comparing your home network's IP to what your phone shows on mobile data. Your phone on cellular data is on a completely different ISP and connection — of course the IPs differ, regardless of whether your home connection has CGNAT or not. That comparison doesn't tell you anything.

**If you are behind CGNAT**, a few options, roughly in order of effort:

1. **Ask your ISP directly:** *"Is my connection assigned a dedicated public IPv4 address, or is it behind CGNAT / a shared IP?"* — sometimes they'll move you off it for free, sometimes it's a paid add-on, sometimes it's simply not offered on your plan tier.
2. **Try IPv6**, which sidesteps CGNAT entirely since it's specifically an IPv4 problem: `ip -6 addr show` on the server — a `2xxx:` or `3xxx:` prefixed address means you likely have real, publicly-routable IPv6. The catch: your friends' ISPs also need working IPv6 for them to reach you this way, which isn't universal.
3. **A tunneling service** (Playit.gg, Cloudflare Tunnel) as a fallback — free, runs on the server itself, and works around CGNAT completely at the cost of a small added latency hop.

The rest of this part assumes you have a working public IP, via whichever of the above got you there.

## Step 2: Tune server.properties

```bash
sudo systemctl stop minecraft
sudo -u minecraft nano /opt/minecraft/server/server.properties
```

```properties
view-distance=8
simulation-distance=6
max-players=10
online-mode=true
white-list=true
```

`view-distance`/`simulation-distance` trimmed down from the defaults (usually 10) keeps CPU load reasonable on modest hardware — raise them later if performance monitoring (Part 6) shows you have headroom to spare. `online-mode=true` keeps Mojang account verification on. `white-list=true` matters specifically because this server is about to become internet-facing — without it, anyone who finds the address (including automated scanners) could attempt to join.

```bash
sudo systemctl start minecraft
```

## Step 3: Add friends to the whitelist

```bash
sudo -u minecraft screen -r mcserver
```

```
whitelist add FriendUsername
```

(repeat per friend, using their exact Minecraft username), then detach with `Ctrl+A` then `D`.

## Step 4: Port forward

In your router's admin page, look for **NAT**, **Virtual Server**, or **Port Forwarding** — often grouped under an "Advanced" section. Forward external **TCP** port `25565` to your server's reserved local IP (from Part 4), port `25565`.

## Step 5: Give friends a stable address

If your public IP is genuinely static, you can skip this and just share the raw IP — it won't change. Most home connections are dynamic, though, meaning it can shift without warning and break everyone's saved server address. Worth knowing: whether your ISP will make it static, and whether that costs anything, is worth asking directly — some offer it free, some charge a recurring fee. **Static vs. dynamic makes essentially no security difference** either way (what actually matters for security is the firewall + whitelist from Part 4, not whether the address itself changes), so this is purely a convenience decision, not a safety one.

If you'd rather not pay for a static IP, **DuckDNS** is a solid free alternative:

1. Go to [duckdns.org](https://www.duckdns.org), sign in with an existing account (GitHub/Google/Reddit/Twitter), and add a subdomain — e.g. `yourname.duckdns.org`. Copy the token shown near the top of the page.
2. On the server:
   ```bash
   mkdir -p ~/duckdns && cd ~/duckdns
   nano duck.sh
   ```
   ```bash
   echo url="https://www.duckdns.org/update?domains=YOURSUBDOMAIN&token=YOURTOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
   ```
   (swap in your real subdomain and token)
3. Test it and schedule it to keep running:
   ```bash
   chmod 700 duck.sh
   ./duck.sh && cat duck.log   # should print "OK"
   sudo apt install cron -y    # if not already installed
   crontab -e                  # your own user, no sudo
   ```
   add:
   ```cron
   */5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
   ```
4. Verify it resolves correctly:
   ```bash
   sudo apt install dnsutils -y   # if `dig` isn't found
   dig +short yourname.duckdns.org
   ```
   should match your public IP from `curl ifconfig.me`.

## Step 6: Test the full connection

From a device genuinely outside your home network (phone on mobile data, WiFi off — or ask a friend), add a server in the Minecraft launcher using your public IP or DuckDNS domain, and try to join.

If it doesn't connect, check in this order: is the server actually running (`sudo systemctl status minecraft`), is the port forward rule saved correctly on the router, and is the player's username actually on the whitelist.

---

**Next: [Part 6 — Maintenance & Troubleshooting](06-maintenance-and-troubleshooting.md)**
