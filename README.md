# Wifi-Mate

A Claude Code skill that sets up your own network gear for you. Send it a photo of your router, access point or firewall. It finds the admin interface, works out how the device loads its data and then does one of two jobs:

1. **Privacy check** (read-only): which devices the box remembers, what your internet provider can see and reach (TR-069, DNS), how old the firmware is, what it logs and how to fix it.
2. **Fix slow Wi-Fi**: backs up the settings, runs a speed test before and after, changes Wi-Fi settings only after you say yes and leaves you a small tool for chat control ("who's on my Wi-Fi?", "reboot the router").

Your own network only. No exploits, no password guessing, no port scans and no passwords in the chat.

> Based on [reverse-engineer-Wi-Fi-router](https://github.com/Bodila51/reverse-engineer-Wi-Fi-router) by Bodila51 (MIT).

---

## Supported devices

| Device | Privacy check | Fix slow Wi-Fi | Notes |
|---|---|---|---|
| **ISP / home router** (all-in-one) | ✅ Full | ✅ Full | This is what the skill was built for |
| **Standalone access point** (TP-Link EAP, Aruba Instant On, Ruckus Unleashed, MikroTik cAP, a router in AP mode) | ⚠️ Partial | ✅ Full | See [Access points](#access-points) |
| **Controller-managed APs** (UniFi, Omada, Meraki) | ⚠️ Partial | ✅ Works, best done through the controller | Point it at the controller, not the AP |
| **Firewall / gateway** (pfSense, OPNsense, FortiGate, Sophos, MikroTik, UniFi gateway) | ⚠️ Partial | ❌ Doesn't apply without a built-in radio | See [Firewalls](#firewalls) |
| **App-only mesh** (eero, Google Nest, some Deco) | ⚠️ Limited | ⚠️ Limited | No web panel, so it only covers what the app and your computer can show |

### Access points

**What works.** The Wi-Fi fix job fits APs very well. Channel, channel width, mode, transmit power and band steering are all AP settings, and the skill checks every one of them against neighboring networks. Backup, speed test before and after, approval before each change and chat control all work the same way. The privacy check still lists connected clients, private-MAC usage, firmware age and logs.

**What doesn't.** An AP in bridge mode doesn't route traffic, give out IP addresses or DNS, or hold the WAN link. The DNS, UPnP, WAN-name and TR-069 parts of the privacy check belong to whatever box sits upstream, so run the check on that box as well.

**Tips**
- The skill looks for the default gateway, and an AP is usually not the gateway. Tell Claude the AP's IP address, or let it find the AP in the router's DHCP lease list.
- For UniFi, Omada and similar systems, point Claude at the **controller**. Settings live there, and anything changed on the AP itself gets overwritten. Controllers usually have an API or a Home Assistant integration, so Claude doesn't have to reverse-engineer anything.
- With several APs, ask it to plan channels for all of them at once so they don't overlap.

### Firewalls

**What works.** Most of the privacy check:
- devices it knows about (DHCP leases, ARP table)
- WAN interfaces and any remote-management channel
- which DNS servers it hands out and who owns them
- UPnP/NAT-PMP status and automatic port forwards
- firmware version and whether you can update it yourself
- which logs exist and what they record

The DNS check and traceroute run from your computer, so they work with any device. Many firewalls also have documented APIs (OPNsense REST, FortiOS REST, MikroTik RouterOS API, the pfSense REST package), so getting in is often easier than on a consumer router.

**What doesn't.**
- **Fix slow Wi-Fi** only changes Wi-Fi settings, so it does nothing on a firewall without a radio. It can still run the speed and bufferbloat test and point to the firewall as the cause (shaping, IDS/IPS load, a missing SQM). It won't change those settings, though.
- **No SSH.** The skill bans SSH and telnet, while firewalls are often managed over SSH or a CLI. If your firewall has no web or API access, you have to relax that rule for your own device yourself.
- **No rule audit.** The skill doesn't review firewall rules, look for an admin page exposed to the internet, check VPN settings or look up firmware CVEs. Those would need extra steps in `SKILL.md`.
- **Home scope.** The rules say "home network only". Use it on a work or company firewall only when you're authorized to.

**Bottom line:** use it as-is for routers and APs. For firewalls it's a good read-only privacy and exposure check, not a hardening tool.

---

## Install

As a Claude Code skill:

```bash
git clone https://github.com/TanimSec/Wifi-Mate ~/.claude/skills/reverse-engineer-wifi-router
```

Then open Claude Code on a computer on the same network, attach a photo of the device (cover the password sticker) and ask:

- "what does my router know about me?"
- "my Wi-Fi is slow, fix it"
- "check my access point at 192.168.1.2" / "check my OPNsense firewall"

If you'd rather not install anything, paste one of these prompts into Claude Code instead:

- [prompts/privacy-check.md](prompts/privacy-check.md)
- [prompts/fix-slow-wifi.md](prompts/fix-slow-wifi.md)

Claude gives you one command that saves your admin password to your system's password store (macOS Keychain, Windows Credential Manager or Linux `secret-tool`). The password never goes into the chat.

## How to fix your privacy

1. Change the admin password from the one on the sticker.
2. Turn on encrypted DNS: Private DNS on Android, secure DNS in Chrome or Firefox, DNS encryption on Windows 11, or the free [1.1.1.1 app](https://one.one.one.one) on iPhone and Mac.
3. Turn on Private Wi-Fi Address on your phone.
4. Add `_nomap` to the end of your Wi-Fi name so [Apple](https://support.apple.com/en-us/102515) and [Google](https://support.google.com/maps/answer/1725632) stop adding it to their location maps.
5. Change your Wi-Fi password if the device remembers clients you don't recognize.

Encrypted DNS hides your lookups but not which servers you connect to. Only a VPN hides that.

## Safety rules the skill follows

- Read-only unless you approve a specific change, and the old and new values are shown first
- Never changes the Wi-Fi name or passwords, never updates firmware or resets, and never reboots without asking
- No exploits, brute-forcing, port scans or SSH/telnet attempts
- Everything shown goes through `scripts/mask.py` to hide public IPs, MAC addresses, serial numbers and SSIDs
- Anything it infers is labeled as inferred

## What's inside

| Path | Purpose |
|---|---|
| `SKILL.md` | Instructions Claude follows for both jobs |
| `prompts/` | The same jobs as copy-paste prompts |
| `scripts/dns-check.sh` | Shows which company answers your DNS lookups. Run it before and after the fix |
| `scripts/mask.py` | Masks public IPs, MACs, serials and SSIDs in any output so screenshots are safe |

## Background

- Nearly 1 billion home devices have the TR-069 remote-management channel. Hils & Böhme, ACNS 2020, [arXiv:2001.02564](https://arxiv.org/abs/2001.02564)
- Researchers pulled the locations of more than 2 billion routers from Apple's Wi-Fi positioning database. Rye & Levin, 2024, [arXiv:2405.14975](https://arxiv.org/abs/2405.14975)

## License

MIT. The original work is copyright (c) 2026 Bodila51. See [LICENSE](LICENSE).
