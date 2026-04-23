# 🔴 Red Team Recon Lab — Metasploitable2
### Portfolio Project #2 | Reconnaissance & Network Scanning

> **Disclaimer:** This lab was conducted in a fully isolated, local virtual environment. All targets are intentionally vulnerable machines designed for educational purposes. No real systems were harmed or accessed without authorization. This project is for educational and portfolio purposes only.

---

## 📋 Table of Contents

1. [Objective](#objective)
2. [Lab Environment](#lab-environment)
3. [Tools Used](#tools-used)
4. [Phase 1 — Connectivity Verification](#phase-1--connectivity-verification)
5. [Phase 2 — Network Scanning with Nmap](#phase-2--network-scanning-with-nmap)
6. [Phase 3 — Reconnaissance with Metasploit](#phase-3--reconnaissance-with-metasploit)
7. [Findings Summary](#findings-summary)
8. [Conclusions & Next Steps](#conclusions--next-steps)

---

## Objective

Perform a structured **reconnaissance and network scanning exercise** against a vulnerable target machine (Metasploitable2) from an attacker machine (Kali Linux) within an isolated lab environment.

The goal is to simulate the **initial phases of a red team engagement**:
- Confirm connectivity to the target
- Enumerate open ports and running services
- Identify service versions and potential attack vectors
- Document findings in a professional report format

---

## Lab Environment

```
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
```

| Role     | OS               | IP Address      |
|----------|------------------|-----------------|
| Attacker | Kali Linux       | 192.168.128.4   |
| Target   | Metasploitable2  | 192.168.128.3   |

> Both machines run in UTM on an isolated host-only network with no internet access.

---

## Tools Used

| Tool        | Version  | Purpose                              |
|-------------|----------|--------------------------------------|
| Nmap        | 7.99     | Port scanning & service enumeration  |
| Metasploit  | Latest   | Auxiliary module scanning            |
| Ping        | Built-in | Connectivity verification            |

---

## Phase 1 — Connectivity Verification

**Objective:** Confirm that the attacker machine can reach the target before any scanning.

**Command:**
```bash
ping -c 4 192.168.128.3
```

**Result:** Target responded to all 4 ICMP packets with ~0.4ms latency, confirming the machines are on the same network and communication is possible.

📸 *Screenshot: `screenshots/01_ping.png`*

---

## Phase 2 — Network Scanning with Nmap

### 2.1 — Basic Port Scan

**Command:**
```bash
nmap 192.168.128.3
```

📸 *Screenshot: `screenshots/02_nmap_basico.png`*

---

### 2.2 — Service Version Detection

**Command:**
```bash
nmap -sV 192.168.128.3
```

📸 *Screenshot: `screenshots/03_nmap_versiones.png`*

---

### 2.3 — Aggressive Scan (OS + Scripts)

**Command:**
```bash
nmap -A 192.168.128.3
```

📸 *Screenshot: `screenshots/04_nmap_agresivo.png`*

---

### 2.4 — Full Scan Output to File

**Command:**
```bash
nmap -A -oN scans/nmap_completo.txt 192.168.128.3
```

Full output saved in [`scans/nmap_completo.txt`](scans/nmap_completo.txt)

---

### 📊 Nmap Results — Open Ports

Scan date: **2026-04-15** | Host status: **UP** | Latency: **0.00040s**

| Port     | State | Service       | Risk Level | Notes                          |
|----------|-------|---------------|------------|--------------------------------|
| 21/tcp   | open  | FTP           | 🔴 High    | Likely vsftpd — known vulns    |
| 22/tcp   | open  | SSH           | 🟡 Medium  | Brute-force vector             |
| 23/tcp   | open  | Telnet        | 🔴 High    | Unencrypted — credentials exposed |
| 25/tcp   | open  | SMTP          | 🟡 Medium  | Mail relay possible            |
| 53/tcp   | open  | DNS           | 🟡 Medium  | Zone transfer possible         |
| 80/tcp   | open  | HTTP          | 🔴 High    | Web app — likely DVWA/phpMyAdmin |
| 111/tcp  | open  | RPCbind       | 🟡 Medium  | NFS enumeration vector         |
| 139/tcp  | open  | NetBIOS-SSN   | 🔴 High    | SMB — known exploits           |
| 445/tcp  | open  | Microsoft-DS  | 🔴 High    | SMB — EternalBlue class vulns  |
| 512/tcp  | open  | exec          | 🔴 High    | rexec — no auth by default     |
| 513/tcp  | open  | login         | 🔴 High    | rlogin — plaintext             |
| 514/tcp  | open  | shell         | 🔴 High    | rsh — no encryption            |
| 1099/tcp | open  | RMI Registry  | 🔴 High    | Java RMI — remote code exec    |
| 1524/tcp | open  | ingreslock    | 🔴 High    | Metasploitable backdoor shell  |
| 2049/tcp | open  | NFS           | 🟡 Medium  | File share enumeration         |
| 2121/tcp | open  | FTP (alt)     | 🟡 Medium  | Secondary FTP service          |
| 3306/tcp | open  | MySQL         | 🔴 High    | DB exposed — default creds     |
| 5432/tcp | open  | PostgreSQL    | 🔴 High    | DB exposed — default creds     |
| 5900/tcp | open  | VNC           | 🔴 High    | Remote desktop — weak auth     |
| 6000/tcp | open  | X11           | 🔴 High    | Display server — no auth       |
| 6667/tcp | open  | IRC           | 🟡 Medium  | UnrealIRCd — backdoor version  |
| 8009/tcp | open  | AJP13         | 🔴 High    | Ghostcat vulnerability (CVE-2020-1938) |
| 8180/tcp | open  | HTTP (alt)    | 🟡 Medium  | Tomcat admin panel             |

> **23 open ports identified** out of 1000 scanned.

---

## Phase 3 — Reconnaissance with Metasploit

All auxiliary modules were run from `msfconsole` on Kali Linux (192.168.128.4).

### 3.1 — TCP Port Scan

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.128.3
set PORTS 1-1000
run
```

📸 *Screenshot: `screenshots/05_msf_portscan.png`*

**Result:** Confirmed open ports consistent with Nmap findings.

---

### 3.2 — SMB Version Enumeration

```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.128.3
run
```

📸 *Screenshot: `screenshots/06_msf_smb.png`*

**Result:** Identified SMB service version, confirming Samba is running — a high-value target for exploitation in future phases.

---

### 3.3 — SSH Version Enumeration

```bash
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.128.3
run
```

📸 *Screenshot: `screenshots/07_msf_ssh.png`*

**Result:** SSH version identified. Useful for determining if known CVEs apply.

---

### 3.4 — FTP Version Enumeration

```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.128.3
run
```

📸 *Screenshot: `screenshots/08_msf_ftp.png`*

**Result:** FTP version identified on port 21. vsftpd versions on Metasploitable2 are known to carry a backdoor vulnerability.

---

## Findings Summary

### Attack Surface Overview

| Category        | Count | Risk    |
|-----------------|-------|---------|
| High-risk ports | 16    | 🔴 High |
| Medium-risk ports | 7   | 🟡 Medium |
| Total open ports | 23   | —       |

### Key Vulnerabilities Identified (for future exploitation phases)

| # | Service     | Port  | Potential CVE / Attack              |
|---|-------------|-------|-------------------------------------|
| 1 | vsftpd FTP  | 21    | CVE-2011-2523 — Backdoor RCE        |
| 2 | Samba SMB   | 139/445 | CVE-2007-2447 — Username map script |
| 3 | UnrealIRCd  | 6667  | CVE-2010-2075 — Backdoor RCE        |
| 4 | Tomcat AJP  | 8009  | CVE-2020-1938 — Ghostcat LFI        |
| 5 | Ingreslock  | 1524  | Direct backdoor shell               |
| 6 | Java RMI    | 1099  | Remote code execution               |
| 7 | VNC         | 5900  | Weak/no authentication              |

---

## Conclusions & Next Steps

### What was accomplished
- Successfully established connectivity between attacker and target in an isolated lab
- Performed comprehensive port scanning using both Nmap and Metasploit auxiliary modules
- Identified **23 open ports** with multiple critical services running
- Built a solid attack surface map for future exploitation phases

### Next Steps (Project #3)
The reconnaissance data collected in this project sets up the next phase:

- **Exploitation** — Use identified CVEs to gain initial access (vsftpd backdoor, Samba usermap script)
- **Post-exploitation** — Privilege escalation and lateral movement
- **Reporting** — Full red team engagement report

---

*Project by Javier Cantillana| Red Team Portfolio | 
