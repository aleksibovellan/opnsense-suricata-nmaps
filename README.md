## opnsense-suricata-nmaps
# OPNsense's Suricata IDS/IPS Detection Rules Against Nmap Scans
v. 1.1 / June 6th 2023 by Aleksi Bovellan

Because there weren't many working alerts against Nmap scans built in OPNSense, or even in Suricata's ET Telemetry Pro ruleset, especially against different types of Nmap scans, I wrote a bundle of my own Suricata rules. These rules react to Nmap scan speeds between -T5-T1, and to fragmented Nmap scans too, but without creating too many false alerts, at least in a personal / home / SoHo network setup. Slower Nmap scan detections take more time to trigger, since the packet rate gets so slow.

Included:

- 3 x Suricata rules to detect most Nmap scans WITHOUT port ranges. (Scan types include at least: -Pn, -sS, -sT, -sU, -A, -f)
- 3 x Suricata rules to detect most Nmap scans WITH more specific, common, or known port targets or ranges. (Scan types include at least: -Pn, -sS, -sT, -sU, -A, -f)
- 4 x Suricata rules against any connection attempts to/from port 4444 TCP/UDP (MetaSploit / MeterPreter / NetCat)

(By the way, if you are running both OPNSense/Suricata and CrowdSec at the same time, CrowdSec bans source IP addresses running Nmap scan speeds down to -T2, but not to for -T1-T0. You can always whitelist your own attacking IP address for testing purposes. CrowdSec also ignores fragmented Nmap scans.)

# NMAP EXAMPLES USING:   Nmap 7.9+ / Kali Linux 2023+	  VS.   OPNsense 23.1+  /  Suricata 6.0+  /  CrowdSec 1.5+  /  FreeBSD 13.1+

- nmap -sS -Pn -T1 -f   ->   DETECTED BY SURICATA
- nmap -sU -Pn -T1   ->    DETECTED BY SURICATA
- nmap -sU -T1 -f   ->   DETECTED BY SURICATA
- nmap -sS -Pn -T2 -f   ->   DETECTED BY SURICATA
- nmap -sS -Pn -T2   ->   DETECTED BY SURICATA
- nmap -sU -Pn -T2   ->   DETECTED BY SURICATA
- nmap -sU -T2 -f   ->   DETECTED BY SURICATA
- nmap -sS -Pn -p 21,22,23,69,80,138,139,140 -T2   ->   DETECTED BY SURICATA
- nmap -sS -Pn -p 21,22,23,69,80,138,139,140 -T2 -f   ->   DETECTED BY SURICATA
- nmap -sS -Pn -p 20-140 -T2   ->   DETECTED BY SURICATA

UNFRAGMENTED NMAP TCP SYN SCANS -T1 OR BELOW: NOT DETECTED WITH THESE RULES, BECAUSE 1 TCP PACKET = EVERY 15 SEC. THIS MINIMIZES FALSE POSITIVE ALERTS. I MIGHT IMPROVE THESE RULES LATER.

- nmap -sS -Pn -T1   ->   NOT DETECTED BY SURICATA 
- nmap -sS -Pn -p 20-140 -T1   ->   NOT DETECTED BY SURICATA
- nmap -sS -Pn -p 21,22,23,69,80,138,139,140 -T1   ->   NOT DETECTED BY SURICATA

# USAGE:

- Save the "local.rules" file, or write all alerts in it into ->  /usr/local/etc/suricata/rules/local.rules
- Reload OPNSense's Web GUI's "INTRUSION DETECTION" ‘RULES’ LIST + APPLY RULES
