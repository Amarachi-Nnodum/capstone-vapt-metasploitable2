
# Enterprise Network Vulnerability Assessment and Penetration Testing

**Project:** Cybersecurity Capstone — Enterprise Network Vulnerability Assessment and Penetration Testing Framework
**Prepared by:** Nnodum Amarachi Augustina
**Target System:** Metasploitable2 (192.168.56.101)
**Date:** November 2026

Evidence referenced throughout this report (screenshots, scan outputs, exported reports) is available in the `/screenshots`, `/scans`, and `/reports` folders of this repository.

---

## 1. Executive Summary

This report documents a full penetration testing engagement conducted against
Metasploitable2, a deliberately vulnerable Linux system, within an isolated
VirtualBox lab environment. The assessment followed the standard penetration
testing lifecycle: network discovery, port and service enumeration,
automated vulnerability scanning, manual exploitation, and remediation
planning.

Eleven critical vulnerabilities were identified using Nmap and OpenVAS,
spanning outdated and intentionally backdoored services. Four vulnerabilities
were actively exploited using the Metasploit Framework, each resulting in
command execution on the target, with three yielding full root access. A
fifth backdoor (Ingreslock) was confirmed exploitable via direct connection
and corroborated by OpenVAS's active check. These findings demonstrate the
severe risk posed by running outdated, unpatched, or intentionally
compromised software on a production network, and underline the importance
of timely patching, service hardening, and OS lifecycle management.

## 2. Scope and Methodology

**Scope:** A single target host, Metasploitable2 (192.168.56.101), running
on an isolated VirtualBox host-only network (192.168.56.0/24), with no
connectivity to the public internet from the target.

**Lab environment:**
- Attacker machine: Kali Linux (192.168.56.1)
- Target machine: Metasploitable2 (192.168.56.101)
- Network: VirtualBox Host-Only Adapter (vboxnet0)

**Tools used:**
- Kali Linux — attacking platform
- Nmap — host discovery, port scanning, service/version detection, NSE
  vulnerability scripts
- OpenVAS/GVM — automated vulnerability scanning and CVSS scoring
- Metasploit Framework — exploitation
- Wireshark — network traffic capture and analysis
- VirtualBox — lab virtualization

**Methodology stages:**
1. Network discovery (Netdiscover)
2. Port scanning and service enumeration (Nmap)
3. Automated vulnerability scanning (OpenVAS)
4. Controlled exploitation (Metasploit)
5. CVSS-based vulnerability classification
6. Remediation planning
7. Reporting

## 3. Network Topology

The lab network consists of the Kali Linux attacker machine and the
Metasploitable2 target, connected via VirtualBox's host-only network
(192.168.56.0/24), isolated from the internet. See
`screenshots/capstone-topology-diagram.png` for the full diagram.

## 4. Discovery and Scanning Results

### 4.1 Host Discovery
Netdiscover was used to identify live hosts on the lab subnet, confirming
Metasploitable2 at 192.168.56.101.
See: `screenshots/netdiscover.png`

### 4.2 Port Scanning and Service Enumeration
A full TCP port scan (`nmap -p- -T4`) followed by service/version detection
(`nmap -sV -sC -p-`) revealed over 25 open ports, including FTP, SSH,
Telnet, SMTP, DNS, HTTP, SMB, NFS, MySQL, PostgreSQL, VNC, IRC, and several
intentionally vulnerable or backdoored services.
See: `scans/01-allports.txt`, `scans/02-services-versions.txt`,
`screenshots/nmap_sV_sC.png`

### 4.3 Nmap NSE Vulnerability Scripts
An NSE vulnerability script scan (`nmap --script vuln`) independently
flagged several critical findings, including the vsftpd 2.3.4 backdoor
(CVE-2011-2523), the UnrealIRCd backdoor, SSL/TLS weaknesses (POODLE,
Logjam, weak Diffie-Hellman groups), and a vulnerable RMI registry
configuration.
See: `scans/04-vulnscripts.txt`, `screenshots/nmap_--script.png`

### 4.4 Traffic Analysis
Wireshark was used to capture live network traffic between Kali and
Metasploitable2 during scanning and exploitation activity, confirming
plaintext protocol exchanges (e.g., unencrypted FTP control traffic).
See: `screenshots/wireshark_ftp_capture.png`

## 5. Vulnerability Assessment Results (OpenVAS)

A full OpenVAS scan ("Full and fast" configuration) was run against the
target, identifying 11 distinct findings, all rated Critical severity
(CVSS 9.8–10.0). Key findings included a TWiki remote code execution
vulnerability, a Distributed Ruby (dRuby) RCE, the Ingreslock backdoor,
an exposed rexec service, passwordless rlogin access, and an
end-of-life operating system (Ubuntu 8.04, EOL since 2013).

Full results are available in `reports/report-c87b5edb-4b0c-4e42-8adc-2daecf12adad.pdf`
and `reports/report-c87b5edb-4b0c-4e42-8adc-2daecf12adad.csv`.
See also: `screenshots/openvas_done.png`, `screenshots/vul_table.png`

## 6. Exploitation Walkthrough

Four vulnerabilities were actively exploited using the Metasploit
Framework. Each exploitation attempt was followed by identity verification
(`getuid`/`whoami`/`id`) to confirm the privilege level obtained.

### 6.1 vsftpd 2.3.4 Backdoor (Port 21)
**Module:** `exploit/unix/ftp/vsftpd_234_backdoor`
**Result:** Backdoor successfully spawned; Meterpreter session opened.
**Access gained:** root
**Evidence:** `screenshots/expliot2.png`

### 6.2 UnrealIRCd 3.2.8.1 Backdoor (Port 6667)
**Module:** `exploit/unix/irc/unreal_ircd_3281_backdoor`
**Result:** Target confirmed vulnerable; backdoor command executed
successfully; Meterpreter session opened.
**Access gained:** root
**Evidence:** `screenshots/spawned_backdoor.png`

### 6.3 distccd Remote Code Execution (Port 3632)
**Module:** `exploit/unix/misc/distcc_exec`
**Result:** Initial attempt with the default payload failed to establish a
session; switching to the `cmd/unix/reverse` payload succeeded, opening a
command shell session.
**Access gained:** daemon
**Evidence:** `screenshots/payroad_exploit.png`

### 6.4 Samba usermap_script Command Execution (Ports 139/445)
**Module:** `exploit/multi/samba/usermap_script`
**Result:** Command shell session opened successfully.
**Access gained:** root
**Evidence:** `screenshots/samba_exploit.png`

### 6.5 Ingreslock Backdoor (Port 1524)
Not exploited via Metasploit; identified and confirmed by OpenVAS's active
check, which sent an `id` command to the open backdoor port and received
`uid=0(root) gid=0(root)` in response, confirming unauthenticated root
access.

## 7. CVSS Vulnerability Classification

A full classification of all 11 identified vulnerabilities, including CVE
references, CVSS scores, severity ratings, and exploitation status, is
provided in `reports/Capstone_CVSS_Classification_Table.xlsx`.

Summary: all 11 findings were rated Critical (CVSS 9.8–10.0). Four were
actively exploited with a 100% success rate against attempted services.

## 8. Remediation Recommendations

Full remediation recommendations for each finding are provided in
`reports/remediation-recommendations.md`. Key priorities, in order:

1. Upgrade the end-of-life operating system — this is the foundational fix,
   since no other patch is effective without vendor security support.
2. Remove and reinstall backdoored services (vsftpd, UnrealIRCd) from
   verified official sources.
3. Disable legacy, unauthenticated remote access services (rexec, rlogin,
   distccd) in favor of SSH.
4. Patch or reconfigure Samba, TWiki, and Apache Tomcat to current secure
   versions.
5. Treat the Ingreslock-infected host as compromised and reimage rather than
   attempt in-place remediation.

## 9. Conclusion

This assessment confirmed that Metasploitable2 contains multiple critical,
easily exploitable vulnerabilities, several of which are intentional
backdoors rather than conventional software flaws. All four actively
targeted exploits succeeded, three yielding immediate root-level access with
no authentication required. These results illustrate the severe
consequences of running outdated software, exposing legacy unauthenticated
services, and failing to apply vendor patches. The remediation steps
outlined in this report, if applied, would eliminate every identified attack
path discovered during this engagement.
