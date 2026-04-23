🔴 Red Team Recon Lab — Metasploitable2
Portfolio Project #2 | Reconnaissance & Network Scanning

Disclaimer: This lab was conducted in a fully isolated, local virtual environment. All targets are intentionally vulnerable machines designed for educational purposes. No real systems were harmed or accessed without authorization. This project is for educational and portfolio purposes only.


📋 Table of Contents

Objective
Lab Environment
Tools Used
Phase 1 — Connectivity Verification
Phase 2 — Network Scanning with Nmap
Phase 3 — Reconnaissance with Metasploit
Findings Summary
Conclusions & Next Steps


Objective
Perform a structured reconnaissance and network scanning exercise against a vulnerable target machine (Metasploitable2) from an attacker machine (Kali Linux) within an isolated lab environment.
The goal is to simulate the initial phases of a red team engagement:

Confirm connectivity to the target
Enumerate open ports and running services
Identify service versions and potential attack vectors
Document findings in a professional report format


Lab Environment
┌─────────────────────────────────────────────────────┐
│                  UTM Hypervisor (macOS)              │
│                                                     │
│   ┌──────────────────┐     ┌──────────────────┐     │
│   │   Kali Linux     │     │  Metasploitable2 │     │
│   │  192.168.128.4   │────▶│  192.168.128.3   │     │
│   │   (Attacker)     │     │    (Target)      │     │
│   └──────────────────┘     └──────────────────┘     │
│                                                     │
│              Isolated Host-Only Network              │
└─────────────────────────────────────────────────────┘
RoleOSIP AddressAttackerKali Linux192.168.128.4TargetMetasploitable2192.168.128.3

Both machines run in UTM on an isolated host-only network with no internet access.


Tools Used
ToolVersionPurposeNmap7.99Port scanning & service enumerationMetasploitLatestAuxiliary module scanningPingBuilt-inConnectivity verification

Phase 1 — Connectivity Verification
Objective: Confirm that the attacker machine can reach the target before any scanning.
Command:
bashping -c 4 192.168.128.3
Result: Target responded to all 4 ICMP packets with ~0.4ms latency, confirming the machines are on the same network and communication is possible.
📸 Screenshot: screenshots/01_ping.png

Phase 2 — Network Scanning with Nmap
2.1 — Basic Port Scan
Command:
bashnmap 192.168.128.3
📸 Screenshot: screenshots/02_nmap_basico.png

2.2 — Service Version Detection
Command:
bashnmap -sV 192.168.128.3
📸 Screenshot: screenshots/03_nmap_versiones.png

2.3 — Aggressive Scan (OS + Scripts)
Command:
bashnmap -A 192.168.128.3
📸 Screenshot: screenshots/04_nmap_agresivo.png

2.4 — Full Scan Output to File
Command:
bashnmap -A -oN scans/nmap_completo.txt 192.168.128.3
Full output saved in scans/nmap_completo.txt

📊 Nmap Results — Open Ports
Scan date: 2026-04-15 | Host status: UP | Latency: 0.00040s
PortStateServiceRisk LevelNotes21/tcpopenFTP🔴 HighLikely vsftpd — known vulns22/tcpopenSSH🟡 MediumBrute-force vector23/tcpopenTelnet🔴 HighUnencrypted — credentials exposed25/tcpopenSMTP🟡 MediumMail relay possible53/tcpopenDNS🟡 MediumZone transfer possible80/tcpopenHTTP🔴 HighWeb app — likely DVWA/phpMyAdmin111/tcpopenRPCbind🟡 MediumNFS enumeration vector139/tcpopenNetBIOS-SSN🔴 HighSMB — known exploits445/tcpopenMicrosoft-DS🔴 HighSMB — EternalBlue class vulns512/tcpopenexec🔴 Highrexec — no auth by default513/tcpopenlogin🔴 Highrlogin — plaintext514/tcpopenshell🔴 Highrsh — no encryption1099/tcpopenRMI Registry🔴 HighJava RMI — remote code exec1524/tcpopeningreslock🔴 HighMetasploitable backdoor shell2049/tcpopenNFS🟡 MediumFile share enumeration2121/tcpopenFTP (alt)🟡 MediumSecondary FTP service3306/tcpopenMySQL🔴 HighDB exposed — default creds5432/tcpopenPostgreSQL🔴 HighDB exposed — default creds5900/tcpopenVNC🔴 HighRemote desktop — weak auth6000/tcpopenX11🔴 HighDisplay server — no auth6667/tcpopenIRC🟡 MediumUnrealIRCd — backdoor version8009/tcpopenAJP13🔴 HighGhostcat vulnerability (CVE-2020-1938)8180/tcpopenHTTP (alt)🟡 MediumTomcat admin panel

23 open ports identified out of 1000 scanned.


Phase 3 — Reconnaissance with Metasploit
All auxiliary modules were run from msfconsole on Kali Linux (192.168.128.4).
3.1 — TCP Port Scan
bashuse auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.128.3
set PORTS 1-1000
run
📸 Screenshot: screenshots/05_msf_portscan.png
Result: Confirmed open ports consistent with Nmap findings.

3.2 — SMB Version Enumeration
bashuse auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.128.3
run
📸 Screenshot: screenshots/06_msf_smb.png
Result: Identified SMB service version, confirming Samba is running — a high-value target for exploitation in future phases.

3.3 — SSH Version Enumeration
bashuse auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.128.3
run
📸 Screenshot: screenshots/07_msf_ssh.png
Result: SSH version identified. Useful for determining if known CVEs apply.

3.4 — FTP Version Enumeration
bashuse auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.128.3
run
📸 Screenshot: screenshots/08_msf_ftp.png
Result: FTP version identified on port 21. vsftpd versions on Metasploitable2 are known to carry a backdoor vulnerability.

Findings Summary
Attack Surface Overview
CategoryCountRiskHigh-risk ports16🔴 HighMedium-risk ports7🟡 MediumTotal open ports23—
Key Vulnerabilities Identified (for future exploitation phases)
#ServicePortPotential CVE / Attack1vsftpd FTP21CVE-2011-2523 — Backdoor RCE2Samba SMB139/445CVE-2007-2447 — Username map script3UnrealIRCd6667CVE-2010-2075 — Backdoor RCE4Tomcat AJP8009CVE-2020-1938 — Ghostcat LFI5Ingreslock1524Direct backdoor shell6Java RMI1099Remote code execution7VNC5900Weak/no authentication

Conclusions & Next Steps
What was accomplished

Successfully established connectivity between attacker and target in an isolated lab
Performed comprehensive port scanning using both Nmap and Metasploit auxiliary modules
Identified 23 open ports with multiple critical services running
Built a solid attack surface map for future exploitation phases

Next Steps (Project #3)
The reconnaissance data collected in this project sets up the next phase:

Exploitation — Use identified CVEs to gain initial access (vsftpd backdoor, Samba usermap script)
Post-exploitation — Privilege escalation and lateral movement
Reporting — Full red team engagement report


Project by Javier Cantillana  | Red Team Portfolio | 
