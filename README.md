# OPNsense's Suricata IDS/IPS NMAP Detection Rules
## v. 1.4.3 / March 24th 2024 by Aleksi Bovellan

Because there weren't many working detection alert rules against NMAP port scans in OPNSense's Suricata IDS/IPS - or even in Suricata's ET Telemetry Pro ruleset - especially against slower NMAP scan speeds like the -T0, I wrote a bundle of my own Suricata detection rules to try to detect and log as many of them as possible. These rules have been tested for 1 year now without any problems. Latest versions tested: OPNsense 24.1.4 and Suricata 7.0.4.

Included:

- Suricata rules to detect most NMAP scans WITHOUT port ranges (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules to detect most NMAP scans WITH more specific, common, or known port targets or ranges (Scan types include at least: -sS, -sT, -sU, -Pn, -f)
- Suricata rules against any connection attempts to and from TCP/UDP port 4444 (MetaSploit / MeterPreter / NetCat / Known Trojan)

These Suricata rules work by looking for specific NMAP packet window sizes, other packet specifications, ports and known NMAP timing intervals. These rules react to NMAP scan speeds between -T5-T0, and to fragmented NMAP scans too, but without creating too many false positive alerts at least in a personal / home / SoHo network setup.

Detecting the slowest NMAP -T0 scans - especially -sU (UDP) or -sT (TCP SYN ACK) versions - can take a LOT of time because their packet rates get so slow, and there is usually existing legit traffic too at the same time. For the NMAP -T0 detection rules to work at all without triggering too may false positives, their port ranges had to be limited to a list of known ports only. However, NMAP -T0 speed scans using -sS or -f options instead do get detected much better than -sT types, because their actual packets are slightly more unique and identifiable. At -T1 or faster NMAP scan speeds, the detection rules start to detect all NMAP scan types much better thanks to the increasing amount of packet traffic, and with higher scan speeds the rules also start to listen to more ports. All of these rules offer some known port identification too for better logging purposes by including the word "KNOWN" for known port detections.

(If running both OPNSense/Suricata and CrowdSec at the same time, CrowdSec bans source IP addresses which are detected running port scans with speeds down to -T2, but not down to -T1-T0. You can always whitelist your own attacking IP address in CrowdSec for testing purposes, or otherwise you might get IP-banned from your own router by CrowdSec. CrowdSec ignores fragmented NMAP scans though.)

## SOME NMAP EXAMPLES:

- sudo nmap -sS -Pn -T0    ->    DETECTED BY SURICATA
- sudo nmap -sT -Pn -T0    ->    MIGHT BE DETECTED BY SURICATA, BUT THIS WOULD TAKE A VERY LONG TIME
- sudo nmap -sU -Pn -T0    ->    MIGHT BE DETECTED BY SURICATA, BUT THIS WOULD TAKE A VERY LONG TIME
- sudo nmap -sS -Pn -T0 -f    ->    DETECTED BY SURICATA
- sudo nmap -sU -T0 -f    ->    DETECTED BY SURICATA
- sudo nmap -sS -Pn -T1    ->    DETECTED BY SURICATA
- sudo nmap -sT -Pn -T1    ->    DETECTED BY SURICATA
- sudo nmap -sU -Pn -T1    ->    DETECTED BY SURICATA
- sudo nmap -sS -Pn -T1 -f    ->    DETECTED BY SURICATA
- sudo nmap -sU -T1 -f    ->    DETECTED BY SURICATA
- sudo nmap -sS -Pn -T4 -A    ->    DETECTED BY SURICATA

## USAGE:

IMPORTANT: If a previous "local.rules" file exists, check for resulting duplicate rule sid numbers in the existing one and this one, and modify them as you wish to not clash.

- Save this "local.rules" file - or write all alerts in it as text - into Suricata's custom rule file -> /usr/local/etc/suricata/rules/local.rules
- Just in case, copy it over the OPNsense's automatically created one too at -> /usr/local/etc/suricata/opnsense.rules/local.rules
- Reload OPNSense's Web GUI's "SERVICES" -> "INTRUSION DETECTION" -> "ADMINISTRATION" -> "RULES" tab's list by clicking the "Apply" button in the bottom of the page
- OPTIONAL: If you want, you can reload the whole Suricata service too just in case

## KNOWN ISSUES

- These rules may react to some legit self-made connection attempts too, which happen to resemble NMAP packets and/or are sent in a too rapid rate to be ignored safely.
- Sometimes by lucky accident, the momentary ephemeral port the computer uses for a specific outbound connection just happens to be 4444, which then gets flagged by these rules for being possible malicious bind/reverse shell activity. However, there doesn't seem to be any better way to detect this notorious and common malware port of 4444 in the earliest possible stage, but by port number basis only for inbound and outbound traffic, so the rule is left as is.
- After loading these rules, expect to see alerts triggered from WAN interface as a result of everyday scanning and probing - legal and illegal. Use "whois IP" and IP traces to find out more about the scanners.


