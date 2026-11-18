# Wifi-Mate

A Claude Code skill for your own network gear: home routers, Wi-Fi access points and firewalls. Give it a photo or a model name and it works out how to get into the device. Then it does one of two jobs.

## 1. Privacy and security check (read-only)

- Every device your router or AP remembers, and which are online
- What your internet provider can see and reach (TR-069 remote management, DNS)
- Firmware age, whether you can update it yourself, and known CVEs for your exact version
- Firewall rule audit: admin page open to the internet, allow-everything rules, port forwards, IPv6 gaps, weak VPN settings and SNMP `public`, ranked High, Medium or Low
- Wi-Fi security on APs: WEP, TKIP, WPS and open networks
- What gets logged
- Step-by-step fixes

## 2. Fix slow Wi-Fi or internet

- Backs up all settings first
- Speed test before and after, including lag under load
- Tunes Wi-Fi: channel, width, power and band steering
- Multiple APs or mesh: one channel plan for all of them, applied one AP at a time
- Firewalls: fixes lag under load (bufferbloat) with SQM (fq_codel, CAKE, Smart Queues)
- Every change only after you say yes
- Leaves you a chat tool for questions like "who's on my Wi-Fi?" or "reboot the AP"

## Supported devices

| Device | Privacy & security | Speed fix |
|---|---|---|
| Home / ISP router | ✅ | ✅ Wi-Fi + SQM where supported |
| Standalone AP (TP-Link EAP, Aruba Instant On, MikroTik cAP, OpenWrt, router in AP mode) | ✅ (no WAN/DNS parts, so check the upstream router too) | ✅ |
| Controller-managed APs (UniFi, Omada, Meraki) | ✅ via the controller | ✅ via the controller |
| OPNsense, OpenWrt, MikroTik | ✅ with rule audit | ✅ SQM |
| UniFi gateway | ✅ with rule audit | ✅ Smart Queues |
| pfSense | ✅ with rule audit | ⚠️ Diagnose plus steps for you (limiters need a firewall rule) |
| FortiGate, Sophos, others | ✅ with rule audit | ⚠️ Diagnose plus steps for you |
| eero, Google Nest, some Deco (app-only) | ⚠️ Limited | ⚠️ Limited |

## Install

```bash
git clone https://github.com/TanimSec/Wifi-Mate ~/.claude/skills/wifi-mate
```

Open Claude Code on a computer on the same network, attach a photo of the device (cover the password sticker) and ask, for example:

- "What does my router know about me?"
- "Audit my OPNsense firewall"
- "My Wi-Fi is slow, I have three UniFi APs"
- "Check my access point at 192.168.1.2"

If you don't want to install it, you can paste a prompt from [prompts/](prompts/) instead.

## How it gets in

It tries these in order:
1. An official API
2. An open-source library or Home Assistant integration
3. The admin page's own JavaScript
4. Read-only SSH, only after you say yes

Passwords and API keys go into your system's password store (Keychain, Windows Credential Manager or `secret-tool`). They never go in the chat.

## Safety rules

- Read-only unless you approve a specific change, with the old and new values shown first
- Never changes Wi-Fi names or passwords, admin users, firewall rules, NAT, VPN or IDS settings
- Never updates firmware or resets the device, and never reboots without asking
- No exploits, password guessing, port scans or telnet
- SSH runs only commands from a fixed read-only list
- All output goes through `scripts/mask.py`, which hides public IPs, MAC addresses, serials, SSIDs, keys and hashes. Config exports are saved to disk and never shown on screen

## Quick privacy wins

1. Change the admin password from the one on the sticker and turn on MFA if the device supports it.
2. Turn on encrypted DNS on each device, or once on the firewall.
3. Turn on Private Wi-Fi Address on phones.
4. Add `_nomap` to the end of your Wi-Fi name.
5. Make sure the admin page can't be reached from the internet.

## Files

| Path | What it does |
|---|---|
| `SKILL.md` | The instructions Claude follows |
| `prompts/` | Copy-paste versions of both jobs |
| `scripts/dns-check.sh` | Shows who answers your DNS lookups |
| `scripts/mask.py` | Masks private details in any output |
