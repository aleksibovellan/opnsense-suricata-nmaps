# OPNsense's Suricata IDS/IPS Detection Rules Against Nmap Scans
## v. 1.4 / June 7th 2023 by Aleksi Bovellan

Because there weren't many working alert rules against Nmap scans built in OPNSense - or even in Suricata's ET Telemetry Pro ruleset - especially against slower Nmap scan speeds like the -T0, I wrote a bundle of my own Suricata rules to try to catch them all.

Included:

- Suricata rules to detect most Nmap scans WITHOUT port ranges. (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules to detect most Nmap scans WITH more specific, common, or known port targets or ranges. (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules against any connection attempts to and from TCP/UDP port 4444 (MetaSploit / MeterPreter / NetCat)

These rules work by looking at specific Nmap packet window sizes, other packet specifications, ports and known Nmap timing intervals. The rules react to Nmap scan speeds between -T5-T0, and to fragmented Nmap scans too, but without creating too many false positive alerts, at least in a personal / home / SoHo network setup. Expect to see some alerts triggered from WAN interface now and then, as the result of everyday scanning and probing.

Detecting the slowest Nmap -T0 scans - especially the UDP version - can take time, a LOT of time, since the packet rates get so slow. Also, for the Nmap -T0 detection rules to work without triggering too may false positives, their port ranges had to be limited to a list of known ports only. But at -T1 or faster Nmap scan speeds, the detection rules target all ports, and offer some known port identification as an added bonus for better logging purposes, with such more specific alert rules having the word "KNOWN" in their descriptions.

(If running both OPNSense/Suricata and CrowdSec at the same time, CrowdSec bans source IP addresses detected running Nmap scan speeds down to -T2, but not to -T1-T0. You can always whitelist your own attacking IP address in CrowdSec for testing purposes. CrowdSec also ignores fragmented Nmap scans.)

## STEALTHY NMAP EXAMPLES USING:   Nmap 7.9+ / Kali Linux 2023+	  VS.   OPNsense 23.1+  /  Suricata 6.0+  /  CrowdSec 1.5+  /  FreeBSD 13.1+

- nmap -sS -Pn -T0    ->    DETECTED BY SURICATA
- nmap -sT -Pn -T0    ->    DETECTED BY SURICATA
- nmap -sU -Pn -T0    ->    MIGHT BE DETECTED BY SURICATA, BUT THIS WOULD TAKE A VERY LONG TIME
- nmap -sS -Pn -T0 -f    ->    DETECTED BY SURICATA
- nmap -sU -T0 -f    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sT -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sU -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T1 -f    ->    DETECTED BY SURICATA
- nmap -sU -T1 -f    ->    DETECTED BY SURICATA

## USAGE:

- Save the "local.rules" file, or write all alerts in it, into Suricata's default location -> /usr/local/etc/suricata/rules/local.rules
- If previous custom rules file existed, check for resulting duplicate rule sid numbers after copy-pasting
- Reload OPNSense's Web GUI's "INTRUSION DETECTION" ‘RULES’ LIST
- APPLY Rules
