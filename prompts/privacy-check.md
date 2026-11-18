# Privacy and security check

Paste it into Claude Code with a photo of your router, access point or firewall (or just its model name):

```
I want to see what my own network device knows about me, what my internet provider can see, and whether the device is set up safely. I've attached a photo of it with the password sticker covered. It's my own device on my own network.

1. From the photo, identify the model and its role: router, access point, controller-managed AP (UniFi/Omada/Meraki) or firewall. Find its admin address. An access point is usually not the gateway, so look for it in the router's client list or ask me. For controller-managed APs, use the controller.
2. Log in the cleanest way: an official API first (I'll create a key in the device's UI), then an open-source library, then the admin page's own JavaScript. Read-only SSH only if I say yes, with read-only commands only. Save any password or key to my system's password store with one command you give me, never in this chat. Never guess passwords.
3. Show me:
   - every device it remembers, and which are online now
   - the internet connection names and any sign of remote management by my provider (TR-069, CWMP, ACS)
   - firmware version, whether I can update it myself, and any known CVEs for it (vendor advisories, NVD, CISA KEV)
   - which DNS servers my devices use and who owns them (whois)
   - ports opened automatically (UPnP) and every port forward
   - where the admin page and SSH can be reached from, especially the internet side
   - for firewalls: a rule audit (allow-everything rules, admin exposed on WAN, IPv6 gaps, weak VPN settings, SNMP public) ranked High/Medium/Low
   - for access points: Wi-Fi security mode per network (WEP, TKIP, WPS, open networks)
   - which logs it keeps and whether they have timestamps
4. From this computer, show my DNS servers and the first hops of a traceroute.

Rules: read-only. Change no setting and don't reboot. If viewing something needs a setting switched, ask me first and switch it back afterwards. No exploits, no port scans, no telnet, never print passwords, keys or tokens. Mask MAC addresses, my public IP, serials and Wi-Fi names in everything you show. Config exports go to a "router" folder in my home folder, never on screen.

Done means a short report with four headings: What my device knows, What my provider can see, Security findings, How to fix it. In the fix part, give the official steps to turn on encrypted DNS on each of my devices (or once on the firewall), how to add _nomap to my Wi-Fi name, the fixed firmware version for any CVE, and exact steps for each High finding. Ask before changing anything.
```
