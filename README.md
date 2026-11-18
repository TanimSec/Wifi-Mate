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

## Step-by-step guide

### Step 1: Get what you need

| What | Why | How |
|---|---|---|
| **Claude Code** | Runs the skill | Install it from [claude.com/claude-code](https://claude.com/claude-code) and sign in |
| **Git** | Downloads this repo | Windows: [git-scm.com](https://git-scm.com/download/win) (also gives you Git Bash). macOS: `xcode-select --install`. Linux: `sudo apt install git` |
| **Python 3** | Runs `mask.py`, which hides private details | Windows: [python.org](https://www.python.org/downloads/) or `winget install Python.Python.3.12`. macOS and Linux usually have it already |
| `whois` (optional) | Shows which company owns a DNS server | Windows: `winget install Microsoft.Sysinternals.Whois`. macOS: built in. Linux: `sudo apt install whois` |
| A computer **on the same network** as the device | The skill talks to the device directly | Wi-Fi or a cable is fine |

### Step 2: Install the skill

**macOS / Linux**

```bash
git clone https://github.com/TanimSec/Wifi-Mate ~/.claude/skills/wifi-mate
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/TanimSec/Wifi-Mate "$env:USERPROFILE\.claude\skills\wifi-mate"
```

To update later, run `git pull` inside that folder.

### Step 3: Check that your computer is ready (optional, 1 minute)

From the `wifi-mate` folder:

```bash
bash scripts/dns-check.sh
```

You should see `Your DNS lookups are answered by: x.x.x.x`. On Windows, run it in **Git Bash**.

```bash
ipconfig | python scripts/mask.py      # Windows
ifconfig | python3 scripts/mask.py    # macOS / Linux
```

Public IPs, IPv6 addresses and the second half of MAC addresses should show up masked.

### Step 4: Take a photo

Take a photo of the device's label: brand, model and, if it's there, the admin address. **Cover the password sticker.** If you can't take a photo, the model name is enough, for example "TP-Link Archer AX55", "OPNsense 25.1" or "UniFi U6 Lite".

### Step 5: Ask Claude

Open Claude Code in any folder (`claude` in a terminal, or the VS Code extension), attach the photo and ask in plain words:

| You want | Ask |
|---|---|
| A privacy check of your router | "What does my router know about me?" |
| A firewall audit | "Audit my OPNsense firewall, read-only" |
| A check of one access point | "Check my access point at 192.168.1.2" |
| Faster Wi-Fi with several APs | "My Wi-Fi is slow, I have three UniFi APs" |
| Less lag during video calls | "My internet lags when someone streams, fix it on my firewall" |
| Only a diagnosis, no changes | Add "just diagnose" to any request |

### Step 6: Log in safely

Claude finds the device and works out how it logs in. When it needs your admin password or an API key:

1. Claude gives you **one command**. Run it in your own terminal, not in the chat.
2. The command saves the password to your system's password store (macOS Keychain, Windows Credential Manager or Linux `secret-tool`).
3. Claude reads it from there. It never sees the password in the chat and never prints it.

**Firewalls with an API (recommended):** create a read-only API key in the device's own UI and save it the same way:
- OPNsense: System > Access > Users > your user > API keys
- FortiGate: System > Administrators > Create New > REST API Admin
- MikroTik RouterOS v7: the REST API runs on the `www-ssl` service
- UniFi, Omada, Meraki: the controller's API or integration settings

**SSH:** only if the device has no API and you say yes. Claude runs read-only commands from a fixed list. Using your SSH key is best.

If login fails once, Claude stops and asks you. It never guesses passwords.

### Step 7: Approve changes (speed fix only)

Before each change, Claude shows the old and new values and waits for your **yes**. Settings are backed up first to:
- `~/router/` on macOS and Linux
- `C:\Users\<you>\router\` on Windows

With several APs, you'll see the whole channel plan as a table first, and it's applied one AP at a time.

### Step 8: Read the report

- **Privacy and security check:** *What my device knows*, *What my provider can see*, *Security findings* (High first) and *How to fix it*
- **Speed fix:** *Fixed* (before and after Mbps and lag), *How to undo* and *What I need from you*

### No install? Use a prompt

Copy one of these into Claude Code (or claude.ai) along with your photo:
- [prompts/privacy-check.md](prompts/privacy-check.md)
- [prompts/fix-slow-wifi.md](prompts/fix-slow-wifi.md)

### Troubleshooting

| Problem | Fix |
|---|---|
| Claude doesn't use the skill | Check that the folder is `~/.claude/skills/wifi-mate` (on Windows, `%USERPROFILE%\.claude\skills\wifi-mate`) and contains `SKILL.md`, then restart Claude Code |
| `bash: command not found` on Windows | Run the script in Git Bash. Claude Code on Windows uses it automatically |
| `install whois to see which company owns it` | Install `whois` (see Step 1), or look up the address on any whois website |
| The admin page doesn't load | Check that you're on the device's network (not a guest network or VPN). Tell Claude the device's IP if you know it |
| An access point isn't found | APs are rarely the gateway. Give Claude its IP from your router's client list |
| The login has a captcha or needs your provider's account | Claude opens a browser so you can log in yourself |
| App-only mesh (eero, Nest) | No web panel exists. Claude checks what the app and your computer can show |

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

## Bonus: Site X-ray

[`site-xray/`](site-xray/) is a second skill that checks a **website** instead of your network. Claude opens the site as different visitors (a German laptop, an American laptop and a British phone, or just your own Chrome) and shows:
- which tracking companies get your data before you click anything, and after "Reject all"
- whether your search word or a typed email (never submitted) leaves the site
- ad companies sharing your ID with each other
- session recording and device fingerprinting

### Site X-ray: step by step

**1. Get what you need:** [Node.js 20+](https://nodejs.org) and Google Chrome.

**2. Install.** Do Step 2 above first, then copy the folder into its own skill and install its two dependencies.

macOS / Linux:
```bash
cp -R ~/.claude/skills/wifi-mate/site-xray ~/.claude/skills/site-xray
cd ~/.claude/skills/site-xray && npm install
```

Windows (PowerShell):
```powershell
Copy-Item -Recurse "$env:USERPROFILE\.claude\skills\wifi-mate\site-xray" "$env:USERPROFILE\.claude\skills\site-xray"
cd "$env:USERPROFILE\.claude\skills\site-xray"; npm install
```

**3. Check it:**
```bash
node scripts/xray.mjs doctor
```
You should see `"nodeOk": true`, `"deps": "ok"` and `"chrome": "found"`.

**4. Optional: visitors in other countries.** This needs a [Gologin](https://gologin.com/cloud-browser/) account. Copy your API token from the API section of the Gologin dashboard, then:
- macOS: run the Keychain command in [site-xray/README.md](site-xray/README.md#install)
- Windows (PowerShell, with the token copied to the clipboard): `[Environment]::SetEnvironmentVariable("GOLOGIN_TOKEN", (Get-Clipboard), "User")`, then open a new terminal
- Linux: add `export GOLOGIN_TOKEN=...` to `~/.bashrc`

Without Gologin, everything still works in `local` mode: one visitor, using your own Chrome and connection.

**5. Ask Claude:**
- "X-ray https://some-news-site.com: what does it collect about me?"
- "Does Reject all actually work on <shop url>?"

Claude opens the report page when it's done.

**Run by hand (no Claude):**
```bash
node scripts/xray.mjs start https://example.com/ --as local --run runs/test
node scripts/xray.mjs look --as local --run runs/test
node scripts/xray.mjs stop --as local --run runs/test
node scripts/xray.mjs analyze --run runs/test
node scripts/xray.mjs report --run runs/test
```
The report is saved to `runs/test/report.html`.

## Files

| Path | What it does |
|---|---|
| `SKILL.md` | The instructions Claude follows |
| `prompts/` | Copy-paste versions of both jobs |
| `scripts/dns-check.sh` | Shows who answers your DNS lookups |
| `scripts/mask.py` | Masks private details in any output |
| `site-xray/` | The Site X-ray skill (website privacy check) |
