## opnsense-suricata-nmaps
# OPNsense's Suricata IDS/IPS Detection Rules Against Nmap Scans
v. 1.3 / June 6th 2023 by Aleksi Bovellan

Because there weren't so many working alerts against Nmap scans built in OPNSense - or even in Suricata's ET Telemetry Pro ruleset - especially against different types of slower Nmap scans, I wrote a bundle of my own Suricata rules to try to catch them all. These rules look for specific Nmap scan packet sizes, other packet specifications and known timing intervals. These rules react to Nmap scan speeds between -T5-T1, and to fragmented Nmap scans too, but without creating too many false alerts, at least in a personal / home / SoHo network setup. See examples below. Expect to see alerts now and then triggered from WAN interface, as a result of everyday scanning and probing. (Detecting the slowest Nmap scans take more time, since the packet rates get so slow.)

Included:

- 3 x Suricata rules to detect most Nmap scans WITHOUT port ranges. (Scan types include at least: -Pn, -sS, -sT, -sU, -A, -f)
- 3 x Suricata rules to detect most Nmap scans WITH more specific, common, or known port targets or ranges. (Scan types include at least: -Pn, -sS, -sT, -sU, -A, -f)
- Bonus: 4 x Suricata rules against any connection attempts to/from port 4444 TCP/UDP (MetaSploit / MeterPreter / NetCat)

(If running both OPNSense/Suricata and CrowdSec at the same time, CrowdSec bans source IP addresses running Nmap scan speeds down to -T2, but not to for -T1-T0. You can always whitelist your own attacking IP address for testing purposes. CrowdSec also ignores fragmented Nmap scans.)

# STEALTHY NMAP EXAMPLES USING:   Nmap 7.9+ / Kali Linux 2023+	  VS.   OPNsense 23.1+  /  Suricata 6.0+  /  CrowdSec 1.5+  /  FreeBSD 13.1+

- nmap -sS -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sT -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sU -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T1 -f    ->    DETECTED BY SURICATA
- nmap -sU -T1 -f    ->    DETECTED BY SURICATA
- nmap -T0 scan types    ->    NOT DETECTED BY SURICATA (TOO SLOW PACKET RATES AT -T0, 1 PACKET / 5 MINUTES)

# USAGE:

- Save the "local.rules" file, or write all alerts in it into ->  /usr/local/etc/suricata/rules/local.rules
- Reload OPNSense's Web GUI's "INTRUSION DETECTION" ‘RULES’ LIST + APPLY RULES
