# Fix slow Wi-Fi or internet

Paste it into Claude Code with a photo of your router, access point(s) or firewall:

```
I attached a photo of my network device (if it shows a password, never repeat or save it). It's my own device on my home network. My Wi-Fi or internet is slow: find out why, fix it, and leave me a way to control the device from this chat. If I say "just diagnose", change nothing.

Done means: all settings backed up in a "router" folder in my home folder (one subfolder per device), a speed test before and after including lag under load, each fix applied only after my yes, a small tool so I can ask here "who's on my Wi-Fi" or "reboot the router/AP", and a report: before vs after (Mbps and lag), what changed, how to undo it.

- First find where it's slow: Wi-Fi, the internet link, or lag under load (bufferbloat).
- Get in the cleanest way: an official API (UniFi/Omada/Meraki controller, OPNsense, FortiOS, MikroTik), an open-source library or Home Assistant integration, or the admin page's own JavaScript. Read-only SSH only after my yes. If login needs a captcha or my provider's account, open a browser and let me log in. If my provider hides settings, tell me where to unlock them.
- Password or API key: give me one terminal command that saves it to my system's password store and read it from there. Never ask for it in chat, never print passwords, keys, tokens or cookies. If login fails once, stop and ask me.
- Wi-Fi: check every setting vs the best my device allows: band, channel and neighbors, width, mode, power, band steering. With several APs or mesh nodes, make one channel plan for all of them (2.4 GHz on 1/6/11 only, non-overlapping 5 GHz), show it as a table, and apply it one AP at a time.
- Firewall or router with bufferbloat: you may set up SQM (fq_codel/CAKE, Smart Queues) at about 85–95% of the measured speed, after my yes. Never touch firewall rules, NAT, VPN or IDS settings. If a fix needs a firewall rule (pfSense limiters), give me the steps instead.
- Show old and new values before each change and read every value back after. Never change Wi-Fi names or passwords, reboot without asking, update firmware, reset, or use exploits. If a form resends the Wi-Fi password, keep it unchanged.
- If the device can't be changed, fix what you can on my computer and write the exact message to send my provider.

End with: Fixed, How to undo, What I need from you.
```
