# OPNsense's Suricata IDS/IPS Nmap Detection Rules
## v. 1.4.2 / June 8th 2023 by Aleksi Bovellan

Because there weren't many working detection alert rules against Nmap port scans in OPNSense - or even in Suricata's ET Telemetry Pro ruleset - especially against slower Nmap scan speeds like the -T0, I wrote a bundle of my own Suricata detection rules to try to detect them all.

Included:

- Suricata rules to detect most Nmap scans WITHOUT port ranges. (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules to detect most Nmap scans WITH more specific, common, or known port targets or ranges. (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules against any connection attempts to and from TCP/UDP port 4444 (MetaSploit / MeterPreter / NetCat / Known Trojan)

These Suricata rules work by looking for specific Nmap packet window sizes, other packet specifications, ports and known Nmap timing intervals. These rules react to Nmap scan speeds between -T5-T0, and to fragmented Nmap scans too, but without creating too many false positive alerts, at least in a personal / home / SoHo network setup.

Detecting the slowest Nmap -T0 scans - especially -sU (UDP) or -sT (TCP SYN ACK) versions - can take time, a LOT of time, since their packet rates get so slow, and there is usually existing legit traffic at the same time. For the Nmap -T0 detection rules to work at all without triggering too may false positives, their port ranges had to be limited to a list of known ports only. However, Nmap -T0 speed scans using -sS or -f options instead do get detected better than -sT types, because their actual packets are slightly more unique and identifiable. At -T1 or faster Nmap scan speeds, the detection rules start to detect all Nmap scan types much better thanks to the increasing amount of packet traffic, and the rules also listen to all ports. All of these rules offer some known port identification too as an added bonus for better logging purposes by including the word "KNOWN" for known port detections.

(If running both OPNSense/Suricata and CrowdSec at the same time, CrowdSec bans source IP addresses detected running Nmap scan speeds down to -T2, but not to -T1-T0. You can always whitelist your own attacking IP address in CrowdSec for testing purposes. CrowdSec also ignores fragmented Nmap scans.)

## SOME NMAP EXAMPLES USING:   Nmap 7.9+ in Kali Linux 2023+	VS. OPNsense 23.1+, Suricata 6.0+, CrowdSec 1.5+, FreeBSD 13.1+

- nmap -sS -Pn -T0    ->    DETECTED BY SURICATA
- nmap -sT -Pn -T0    ->    MIGHT BE DETECTED BY SURICATA, BUT THIS WOULD TAKE A VERY LONG TIME
- nmap -sU -Pn -T0    ->    MIGHT BE DETECTED BY SURICATA, BUT THIS WOULD TAKE A VERY LONG TIME
- nmap -sS -Pn -T0 -f    ->    DETECTED BY SURICATA
- nmap -sU -T0 -f    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sT -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sU -Pn -T1    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T1 -f    ->    DETECTED BY SURICATA
- nmap -sU -T1 -f    ->    DETECTED BY SURICATA
- nmap -sS -Pn -T4 -A    ->    DETECTED BY SURICATA

## USAGE:

- Save this "local.rules" file, or write all alerts in it, into Suricata's custom rule file -> /usr/local/etc/suricata/rules/local.rules
- If a previous "local.rules" file existed, check for resulting duplicate rule sid numbers after copy-pasting these rules into it
- Reload OPNSense's Web GUI's "INTRUSION DETECTION" ‘RULES’ LIST
- APPLY Rules

## KNOWN ISSUES

- After loading and applying these detection rules in OPNSense's Suricata, the following types of warnings may appear in Suricata's log, but will cause no problems: [100770] <Warning> -- [ERRCODE: SC_WARN_POOR_RULE(276)] - rule 1000011: SYN-only to port(s) 0:20 w/o direction specified, disabling for toclient direction
- These rules may react to some legit self-made connection attempts, which happen to resemble Nmap packets and/or are sent in a too rapid rate to be ignored safely
- Expect to see some alerts triggered from WAN interface as a result of everyday scanning and probing - legal and illegal. Use "whois IP" and IP traces to find out more about the scanners. Luckily, it's possible to request the removal of your IP address from some of those apparently legit scanners, like for example: https://www.recyber.net/opt-out , http://scanner.openportstats.com/delete , and email to: abuse @ ipvolume.net
