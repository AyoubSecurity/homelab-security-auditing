# Project 2 — Network Security Audit

A practical network security audit performed against an isolated VirtualBox laboratory network.

The objective of this project was to simulate a small-business network security assessment covering asset discovery, service enumeration, protocol security, vulnerability identification, evidence collection, remediation, and verification.

## Objectives

* Discover active hosts on the network
* Identify exposed TCP and UDP services
* Enumerate service versions and configurations
* Assess SMB security
* Identify known vulnerabilities
* Review basic host firewall and service configuration
* Document security findings and their impact
* Apply selected remediation measures
* Re-test the environment and verify remediation
* Produce a professional security audit report

## Lab Environment

| System                    |    IP Address | Role                        |
| ------------------------- | ------------: | --------------------------- |
| Kali Linux                | `10.10.10.10` | Security audit workstation  |
| Windows 7                 | `10.10.10.20` | Windows client / target     |
| Ubuntu Server             | `10.10.10.30` | Linux server / target       |
| VirtualBox Host Interface |  `10.10.10.1` | Host-side network interface |

Network:

```text
10.10.10.0/27
```

### Topology

```text
                         Host Machine
                              |
                         VirtualBox
                              |
                         10.10.10.0/27
                              |
          +-------------------+-------------------+
          |                   |                   |
   Kali Linux            Windows 7          Ubuntu Server
   10.10.10.10           10.10.10.20        10.10.10.30
     Auditor                Target               Target
```

## Tools

* Kali Linux
* Nmap 7.99
* Wireshark / tcpdump where applicable
* Windows built-in networking and firewall tools
* Ubuntu `ss`
* UFW
* Apache
* OpenSSH

## Methodology

The assessment followed this workflow:

```text
Host Discovery
      ↓
Asset Inventory
      ↓
TCP/UDP Enumeration
      ↓
Service & Version Detection
      ↓
Protocol Security Assessment
      ↓
Vulnerability Assessment
      ↓
Configuration Review
      ↓
Findings & Risk Assessment
      ↓
Remediation
      ↓
Re-Test
      ↓
Final Report
```

## Assessment Activities

### 1. Host Discovery

Nmap was used to identify active systems on the isolated network.

```bash
sudo nmap -sn 10.10.10.0/27
```

Discovered hosts included:

* `10.10.10.10` — Kali
* `10.10.10.20` — Windows
* `10.10.10.30` — Ubuntu
* `10.10.10.1` — VirtualBox host interface

Raw discovery results are stored in:

```text
scans/discovery.txt
```

### 2. TCP Enumeration

A full TCP port scan was performed against the Windows and Ubuntu targets.

```bash
sudo nmap -p- --min-rate 1000 10.10.10.20 10.10.10.30
```

The main exposed services identified were:

**Windows**

```text
135/tcp   MSRPC
139/tcp   NetBIOS
445/tcp   SMB
49153-49156/tcp   MSRPC
```

**Ubuntu**

```text
22/tcp    SSH
80/tcp    HTTP
```

### 3. UDP Enumeration

The top 100 UDP ports were assessed.

The Windows host exposed:

```text
137/udp   NetBIOS Name Service
```

No open top-100 UDP ports were identified on the Ubuntu server.

### 4. SMB Security Assessment

The Windows SMB implementation was assessed using:

```bash
sudo nmap -p139,445 --script smb-protocols,smb-security-mode 10.10.10.20
```

The initial assessment identified:

* SMBv1 enabled
* SMBv2 dialects available
* SMB message signing disabled

### 5. Vulnerability Assessment

Nmap vulnerability scripts were used against the Windows and Ubuntu systems.

The Windows assessment identified:

```text
MS17-010
CVE-2017-0143
```

The initial scan reported the Windows system as vulnerable to the SMBv1 remote-code-execution vulnerability.

The Ubuntu vulnerability scan did not identify a confirmed vulnerability through the Nmap checks performed.

## Findings

| ID      | Finding                             | Host          | Severity | Status                         |
| ------- | ----------------------------------- | ------------- | -------- | ------------------------------ |
| NET-001 | SMBv1 / MS17-010 exposure           | `10.10.10.20` | Critical | Remediated                     |
| NET-002 | SMB message signing disabled        | `10.10.10.20` | High     | Not verified after remediation |
| NET-003 | SMBv1 protocol enabled              | `10.10.10.20` | High     | Remediated                     |
| NET-004 | Default Apache HTTP service exposed | `10.10.10.30` | Low      | Configuration observation      |

## Remediation

### Windows SMBv1

SMBv1 was disabled on the Windows laboratory system.

Before:

```text
NT LM 0.12 (SMBv1)
```

After remediation:

```text
2.0.2
2.1
```

SMBv1 was no longer reported by the follow-up Nmap scan.

### MS17-010

The initial assessment reported MS17-010/CVE-2017-0143 as vulnerable.

After SMBv1 was disabled, a targeted follow-up scan:

```bash
sudo nmap --script smb-vuln-ms17-010 -p445 10.10.10.20
```

did not report the vulnerability.

The laboratory verification therefore demonstrated removal of the vulnerable SMBv1 attack surface. In a real environment, OS patch level and vendor security updates would also need to be verified.

### SMB Signing

SMB message signing was initially reported as disabled.

The post-remediation Nmap `smb-security-mode` scan did not return a signing state, so this project does **not** claim that SMB signing was successfully remediated.

### Ubuntu Apache

Apache was identified on TCP/80 and was serving the default Ubuntu Apache page.

This was treated as a low-severity configuration/exposure observation rather than a confirmed Apache vulnerability.

The appropriate remediation depends on whether HTTP is required:

* Restrict HTTP access through firewall/network policy if required.
* Replace the default page with the intended application.
* Harden the web server.
* Disable the service if it is genuinely unnecessary.

## Evidence

Raw assessment evidence is preserved in:

```text
scans/
```

The repository intentionally retains raw scan output rather than only screenshots so that findings can be independently reviewed.

Important evidence includes:

```text
scans/discovery.txt
scans/tcp-all-ports.txt
scans/udp-services.txt
scans/tcp-services-ubuntu-server.txt
scans/ubuntu-services-http-ssh.txt
scans/windows-smb.txt
scans/vulnerability-windows.txt
scans/vulnerability-ubuntu-server.txt
scans/before/windows-smb.txt
scans/before/windows-vuln.txt
scans/after/windows-smb.txt
scans/after/ms17-010.txt
```

## Report

The complete professional audit report is available here:

```text
report/Network_Security_Audit_Report.pdf
```

The report contains:

* Executive summary
* Scope
* Network architecture
* Asset inventory
* Methodology
* Service inventory
* Security findings
* Risk assessment
* Evidence
* Remediation
* Before/after verification
* Recommendations
* Limitations
* Conclusion

## Lessons Learned

This project provided hands-on practice with:

* Network reconnaissance
* Nmap host discovery
* Full TCP enumeration
* UDP enumeration
* Service/version detection
* SMB security assessment
* Vulnerability identification
* Security evidence collection
* Risk-based finding documentation
* Windows security hardening
* Remediation verification
* Professional security-report writing

## Limitations

This was performed in an isolated VirtualBox laboratory and does not represent a production penetration test or complete enterprise vulnerability assessment.

Nmap findings were treated as assessment evidence and were not automatically considered confirmed vulnerabilities without appropriate interpretation.

The project focused on network exposure and selected host-security controls rather than comprehensive authenticated vulnerability scanning.

## Future Improvements

Potential extensions include:

* SNMP security auditing
* Network segmentation assessment
* DNS/DHCP security assessment
* Linux SSH hardening assessment
* Automated audit evidence collection
* Centralized vulnerability tracking
* CVSS-based risk scoring
* Automated PDF report generation
* Network audit automation using Python
* Integration with vulnerability scanners such as OpenVAS/Greenbone

## Disclaimer

All security testing in this project was performed against systems owned and controlled by the author inside an isolated laboratory environment.

Do not perform equivalent scanning or security testing against systems or networks without explicit authorization.
