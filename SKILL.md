---
name: wifi-mate
description: Check and tune your own network gear - home or ISP router, Wi-Fi access point (standalone or UniFi/Omada/Meraki controller), or firewall/gateway (pfSense, OPNsense, OpenWrt, MikroTik, FortiGate, Sophos, UniFi gateway) - starting from a photo or model name. Two jobs - a read-only privacy and security check (every device it remembers, what your internet provider can see and reach through TR-069 and DNS, firewall rule audit, admin exposure, firmware age and known CVEs, logs, how to fix it) and a slow Wi-Fi/internet fix (backup, speed and lag test, multi-AP channel plan, SQM on firewalls, changes only after your yes, chat control). Use when someone asks what their router, AP or firewall knows or exposes, about router privacy, ISP remote access or firewall rules, or says their Wi-Fi or internet is slow.
---

# Wifi-Mate

Work only on network gear the user owns or is authorized to manage. Start from the photo, model name or address they give.

Pick the job from what they ask:
- "what does my router/firewall know", privacy, provider, TR-069, "is my firewall OK", rules, exposure, CVEs: **Job 1, privacy and security check** (read-only)
- slow Wi-Fi, slow internet, lag, drops, "control my router": **Job 2, fix slow Wi-Fi / internet**
- not sure: ask which one, or do Job 1 first

If the network is not obviously a home network (company names, domain-joined machines, many VLANs, an office photo), ask once whether the user is authorized to manage this device before going further.

## Rules for every job

- If the photo shows a password, never repeat or save it.
- Credentials: give the user one terminal command that saves the admin password or API key to the system password store (macOS Keychain, Windows Credential Manager, Linux secret-tool), and read it from there. For SSH, prefer the user's existing SSH key; otherwise read the password from the store. Never ask for it in chat, never print passwords, keys, tokens or cookies. If login fails once, stop and ask.
- Never guess or brute-force passwords, never use exploits, no telnet, no port scans of anything. Nothing outside the local network except DNS, whois, traceroute, speed tests and public advisory lookups (vendor sites, NVD, CISA KEV).
- Never change Wi-Fi names or passwords, admin users, firewall rules, NAT or VPN settings. Never update firmware, reset, or reboot without asking.
- Pipe anything you show through `scripts/mask.py` so public IPs, the second half of MAC addresses, serial numbers, Wi-Fi names, keys and hashes are masked. Config exports and backups go to disk only, never to the screen.
- Report only what you observed. Mark anything inferred as inferred.

## Read-only SSH

SSH is allowed only on the user's own device, only after they say yes in this chat, and only for reading. Log in with their key or the password from the store.

- Run one command at a time, from the list below or an equivalent `show`/`get`/`print`/`cat` command on the same platform. Show the command before running it.
- Never run anything that changes state: `set`, `config`, `configure`, `edit`, `delete`, `unset`, `add`, `remove`, `commit`, `save`, `write`, `apply`, `uci set`, `uci commit`, `rm`, `mv`, `reboot`, `reset`, `upgrade`, `execute`, `/system ...` actions, service restarts, package installs, or editors. No `diagnose` command that changes debug levels or flushes anything.
- pfSense shows a console menu on SSH: choose `8` (Shell), never any other number.
- If a command needs elevated mode or confirmation, stop and ask.

| Platform | Read-only commands |
|---|---|
| pfSense / OPNsense | `pfctl -sr`, `pfctl -sn`, `pfctl -si`, `sockstat -4l`, `sockstat -6l`, `ifconfig`, `netstat -rn`, `arp -an`, `uname -a`, `cat /etc/version`, `clog`/`tail` on `/var/log/*` (summarize only), config: copy `/conf/config.xml` to `~/router/` with `scp`, never print it |
| OpenWrt | `uci show network`, `uci show firewall`, `uci show wireless` (masked), `uci show dhcp`, `iw dev`, `iwinfo`, `nft list ruleset` or `iptables-save`, `ubus call system board`, `logread` (summarize), `cat /proc/net/arp` |
| MikroTik RouterOS | `/system resource print`, `/system routerboard print`, `/ip service print`, `/ip firewall filter print`, `/ip firewall nat print`, `/ipv6 firewall filter print`, `/ip upnp print`, `/ip dns print`, `/ip dhcp-server lease print`, `/interface wifi registration-table print` (v7) or `/interface wireless registration-table print` (v6), `/export terse` (v7 hides secrets by default; on v6 use `/export hide-sensitive`) |
| FortiGate (FortiOS) | `get system status`, `show system interface`, `show firewall policy`, `show firewall vip`, `show system admin`, `show vpn ipsec phase1-interface`, `get system performance status`, `get system arp`, `show system dns` |
| UniFi / EdgeOS / VyOS | `info`, `show configuration`, `show interfaces`, `show nat rules`, `show firewall` |
| Generic Linux AP | `iw dev`, `iw dev <if> station dump`, `iw dev <if> scan` (only if the radio supports it without disconnecting clients), `ip addr`, `ip route` |

## Getting in

1. **Identify the device** from the photo or name: brand, model, ISP-branded or not, and its role:
   - **router**: all-in-one box that is the default gateway
   - **access point**: radios only, usually bridged. Not the default gateway
   - **controller-managed AP**: UniFi, Omada, Meraki, Aruba Central, Ruckus. Settings live in the controller
   - **firewall/gateway**: pfSense, OPNsense, OpenWrt, MikroTik, FortiGate, Sophos, UniFi gateway and similar
   - **app-only mesh**: eero, Google Nest, some Deco. Say what the app can show and do what you can from the computer
2. **Find the address.** Gateway: `route -n get default` (macOS), `ip route` (Linux), `ipconfig` (Windows). An AP is usually not the gateway: ask for its IP, or find it in the router's DHCP lease or client list by name or vendor MAC prefix. For controller-managed APs, find and use the controller (UniFi Network app or console, Omada controller, Meraki dashboard), not the AP.
3. **Use the cleanest way in, in this order:**
   1. An official documented API: OPNsense API (key and secret from System > Access > Users), FortiOS REST API (token from a REST API admin), MikroTik REST (RouterOS v7) or RouterOS API, Omada OpenAPI, Meraki Dashboard API, the UniFi Network API or the local endpoints its app uses. The user creates any key in the device's own UI and saves it to the password store.
   2. An open-source library or Home Assistant integration for the model.
   3. The admin page's own JavaScript: read the login page and its scripts to learn the login flow (tokens, hashing, cookies), then the internal requests (ajax, cgi, json, asp) the pages use.
   4. Read-only SSH (see above), after the user's yes.
   - If login needs a captcha, MFA or the provider's account, open a browser and let the user log in. If the provider hides settings, tell the user where to unlock them.
4. Save a short map of what you found (device role, address, how you got in, useful endpoints or commands) to `~/router/api-map.md`. With several devices, add one section per device.

## Job 1: privacy and security check (read-only)

Change nothing. If seeing something needs a setting switched (for example a log type dropdown), ask first, note the old value and switch it back afterwards.

### All devices

- every client it remembers (online and offline), names, and how many use private (randomized) MAC addresses (locally administered bit set)
- firmware version and build date, whether the user can update it themselves, and whether a config backup exists
- which logs exist, whether they have timestamps and remote syslog, and what kinds of events they record (summarize, never dump whole logs)
- admin access: which interfaces and protocols (HTTP, HTTPS, SSH, telnet, SNMP) the management page listens on, whether HTTP is unencrypted, default admin username, MFA if supported

### Router and firewall (the box that holds the internet link)

- WAN connection names and any sign of provider remote management: TR-069, CWMP, ACS, "management server" pages, connection names like `TR069_INTERNET`
- DNS servers it uses and hands out, and who owns them (`whois`), DNS-over-TLS/HTTPS if configured, DNS rebinding protection
- UPnP / NAT-PMP on or off, and any automatic port mappings

### Firewall rule audit (firewalls and routers that expose rules)

Read the rules from the API, an SSH `show`/`print` command, or a config export saved to `~/router/`. Never test from outside and never port-scan. Report each finding as **High / Medium / Low** with the rule or setting, why it matters and the fix:

- **Admin exposed to the internet**: web UI, SSH, telnet, SNMP or API allowed on WAN (FortiOS `allowaccess` on the WAN interface, RouterOS `/ip service` with no `address` limit plus an input chain that accepts WAN, pfSense/OPNsense WAN rules to "This Firewall" or anti-lockout on WAN). High.
- **Allow-everything rules**: source any, destination any, port any, especially WAN to LAN. High.
- **Port forwards / VIPs / NAT**: list each one with its target. Flag forwards to admin ports (22, 23, 3389, 445, 5900, 8080, 8443), forwards with source "any" that could be restricted, and forwards to devices that no longer exist.
- **No default deny** on WAN input or forward, or the last rule accepts.
- **IPv6 gaps**: IPv4 locked down but IPv6 has allow rules or no filtering.
- **Logging**: deny rules not logged, logging off entirely.
- **VPN**: PPTP enabled, IPsec/IKE with DES, 3DES, MD5 or SHA1, or DH groups below 14, aggressive mode with pre-shared keys, WireGuard or OpenVPN peers the user doesn't recognize.
- **SNMP**: v1/v2c with community `public` or `private`.
- **Segmentation**: IoT or guest networks that can reach the LAN or the admin page.
- **Shadowed or dead rules**: disabled or never-hit rules with a broad scope.

Mark findings you inferred from a partial view (for example rules you couldn't read) as inferred.

### Firmware CVE lookup

From the exact model and firmware version, check public sources only: the vendor's security advisory page (Fortinet PSIRT, Netgate and OPNsense security advisories, MikroTik changelogs and blog, Ubiquiti community security advisories, TP-Link, ASUS, Netgear security pages), NVD (`https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=<vendor>%20<model>`), and the CISA Known Exploited Vulnerabilities catalog. For each match give the CVE ID, severity, whether it's known-exploited, whether the user's version is in the affected range, and the first fixed version. If a range is ambiguous, say "possibly affected". Never download or run exploit code.

### Access points

An AP in bridge mode doesn't route traffic or give out DNS, so skip the WAN, DNS, UPnP and TR-069 parts and say they belong to the upstream router or firewall. Offer to check that device too. Still collect clients, private-MAC count, firmware, CVEs, admin access, logs and SSIDs (masked) with their security mode: flag WEP, WPA (TKIP), WPS on, open networks without client isolation, and WPA2/WPA3 transition mode where WPA3-only would work.

### Check from this computer

- `scripts/dns-check.sh`: which company answers this computer's DNS lookups (on Windows without bash: `nslookup whoami.akamai.net`, then `whois` or an online lookup of the address it returns)
- `traceroute -m 4 1.1.1.1` (`tracert -h 4 1.1.1.1` on Windows): the first hop after the router is the provider's network; `100.64.0.0/10` means carrier-grade NAT

Optional, only if the user says yes: an Android TV / Google TV box on the same network with ADB debugging on. Read per-app network usage with `adb shell dumpsys netstats`, map UIDs to packages with `adb shell cmd package list packages -U`, and point out provider apps that send data while the TV is asleep. Read-only.

### Report

Use these headings: **What my device knows**, **What my provider can see**, **Security findings** (rule audit and CVEs, High first), **How to fix it**. Keep it honest:
- A home router does not store browsing history. The provider sees the sites you look up because DNS goes through it by default.
- TR-069 means the provider *can* remotely read and change the router. Don't claim it is spying.
- Encrypted DNS hides DNS lookups from the provider, not which servers you connect to. Only a VPN hides that.
- A CVE match is only a risk if the vulnerable feature is in use. Say which feature it needs and whether it's on.

Fix steps (official sources, ask before changing anything; firewall rule fixes are steps for the user, never applied by you):
- Encrypted DNS: Android Settings > Network & internet > Private DNS (https://support.google.com/android/answer/9654714); Chrome Settings > Privacy and security > Security > Use secure DNS (https://support.google.com/chrome/answer/10468685); Firefox Settings > Privacy & Security > DNS over HTTPS (https://support.mozilla.org/kb/dns-over-https); Windows 11 DNS encryption (https://learn.microsoft.com/windows-server/networking/dns/doh-client-support); iPhone and Mac have no built-in switch, use Cloudflare's free 1.1.1.1 app in DNS-only mode (https://one.one.one.one). On a firewall, DNS-over-TLS can be set once for the whole network (Unbound on pfSense/OPNsense, `https-dns-proxy` on OpenWrt, `/ip dns set use-doh-server` on RouterOS v7). Run `scripts/dns-check.sh` again after.
- Private Wi-Fi Address on phones (iPhone: Settings > Wi-Fi > (i); Android: Wi-Fi network > Privacy > randomized MAC).
- Add `_nomap` to the end of the Wi-Fi name so Apple (https://support.apple.com/en-us/102515) and Google (https://support.google.com/maps/answer/1725632) stop adding it to their location databases. It doesn't work for hidden networks, and neither company says it deletes existing entries. Renaming is the user's own step.
- Change the admin password from the one on the sticker, turn on MFA where supported, and limit admin access to the LAN.
- Change the Wi-Fi password if the device remembers clients the user doesn't recognize.
- For each High finding: the exact menu path or command to fix it, and how to check it worked.
- For CVEs: the fixed firmware version and the vendor's download or update page. The user updates; you don't.

## Job 2: fix slow Wi-Fi / internet

If the user says "just diagnose", change nothing.

Done means:
- all settings backed up in a `router` folder in the user's home folder (one subfolder per device)
- a speed test before and after, including lag under load. Use `networkQuality` on macOS, the Ookla `speedtest` CLI or https://speed.cloudflare.com elsewhere, and ping `1.1.1.1` while a download runs if the tool doesn't report loaded latency
- each fix applied only after the user's yes, with old and new values shown first and every value read back after
- a small tool so the user can ask in chat "who's on my Wi-Fi" or "reboot the router/AP"
- a report: before X Mbps / Y ms under load vs after, what changed, how to undo it

First find where the slowness is: Wi-Fi (fast wired but slow wireless), the internet link (slow even wired), or bufferbloat (fine when idle, lag spikes under load).

### Wi-Fi: router or single AP

Check every Wi-Fi setting against the best the device allows: band, channel and neighbors on it, width, mode, power, band steering, minimum data rate, airtime fairness. Wi-Fi settings only. If a form resends the Wi-Fi password, keep it unchanged.

### Wi-Fi: several APs or mesh (channel plan)

1. List every AP and radio, with its current channel, width and power, and its client count. Use the controller if there is one.
2. Collect what each AP hears: the AP's or controller's neighbor/RF scan if available. Otherwise scan from this computer next to each AP (`netsh wlan show networks mode=bssid` on Windows, `nmcli -f all dev wifi list` on Linux, `system_profiler SPAirPortDataType` on macOS).
3. Plan:
   - 2.4 GHz: only channels 1, 6 and 11 (1, 5, 9 and 13 where allowed), 20 MHz, so that neighboring APs don't share a channel
   - 5 GHz: non-overlapping channels, 40 or 80 MHz depending on how crowded the area is. Use DFS channels only if every client and the AP support them, and warn that radar events move the AP
   - 6 GHz: leave it on automatic unless something is wrong
   - Power: lower 2.4 GHz than 5 GHz. Turn power down on APs that overlap heavily rather than up
   - If the controller has an automatic channel plan, compare it with yours and recommend one
4. Show the whole plan as a table (AP, band, old, new), then apply it **one AP at a time** after the user's yes, read the values back, check clients reconnect, and run the speed test near that AP before moving on.

### Internet link and bufferbloat (firewalls and routers)

- Measure idle and loaded latency. Above about 30 ms extra under load means bufferbloat.
- On a firewall, also check CPU while loaded (IDS/IPS such as Suricata or Snort, deep packet inspection, VPN encryption) and any existing shaper or limiter that is set wrong.
- **SQM is the only setting you may change on a firewall**, after the user's yes. Set the speeds to about 85–95% of the measured line rate, then re-test and adjust:
  - OPNsense: Firewall > Shaper > Pipes with FQ-CoDel (download and upload pipes), plus the shaper's own rules. These are shaper rules, not firewall rules
  - OpenWrt: `luci-app-sqm` with `cake`. Installing the package is a change: ask first
  - MikroTik RouterOS v7: queue type `cake` with a simple queue or queue tree on the WAN
  - UniFi gateway: turn on Smart Queues with the measured speeds
  - pfSense: limiters with FQ_CODEL need a floating firewall rule. That is a firewall rule, so give the user the exact steps instead of applying it
  - FortiGate, Sophos and others without fq_codel/CAKE: diagnose only, and give the user the vendor's shaping steps
- Never touch firewall rules, NAT, VPN or IDS settings. If IDS/IPS is the bottleneck, explain the trade-off and let the user decide.

If the device can't be changed, fix what you can on the computer and write the exact message to send the provider.

End with: **Fixed**, **How to undo**, **What I need from you**.
