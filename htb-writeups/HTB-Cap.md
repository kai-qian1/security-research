
# HTB Cap - Security Analysis & Defensive Review

**Hack The Box | Easy Linux Machine**

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)
![OS](https://img.shields.io/badge/OS-Linux-blue)
![Focus](https://img.shields.io/badge/Focus-IDOR%20%7C%20PCAP%20%7C%20Linux%20Capabilities-orange)

---

## Machine Information

| Item              | Value                                      |
|-------------------|--------------------------------------------|
| Platform          | Hack The Box                              |
| Machine           | **Cap**                                   |
| Difficulty        | Easy                                      |
| Operating System  | Linux                                     |
| Focus Areas       | IDOR, PCAP Exposure, Credential Leakage, Linux Capabilities |
| Analysis Type     | Security Analysis & Defensive Review      |

---

## Executive Summary

The Cap machine demonstrates how multiple common security weaknesses can chain together to achieve full system compromise.

**Attack Chain:**

**IDOR → Unauthorized PCAP Access → Cleartext FTP Credentials → SSH Login → Python cap_setuid Abuse → Root**

---

## Attack Path Overview

```text
IDOR
  ↓
Unauthorized PCAP File Access (/data/0)
  ↓
Wireshark Analysis → Cleartext FTP Credentials
  ↓
SSH Login as nathan
  ↓
Abuse cap_setuid on Python 3.8
  ↓
Root Shell

Vulnerability Analysis1. Insecure Direct Object Reference (IDOR)Description:
The web dashboard exposes packet capture files using predictable numeric IDs (/data/1, /data/2, ...).Exploitation:
Changing the ID to 0 allows downloading 0.pcap.Root Cause:
No server-side ownership validation — knowing the object ID is enough for access.Defense Recommendations:Implement proper object-level authorization checks
Use UUIDs instead of sequential IDs
Add access control middleware and audit logging


2. Sensitive Data Exposure (PCAP File)Description:
The downloaded PCAP contains cleartext FTP credentials.Credentials:Username: nathan
Password: Buck3tH4TF0RM3!

Root Cause:
Use of insecure plaintext FTP protocol.Defense Recommendations:Replace FTP with SFTP / SCP / FTPS
Enforce TLS encryption for all traffic
Implement credential rotation policies

3. Weak Credential ManagementIssue:
The same credentials are reused across FTP and SSH services.Defense Recommendations:Use unique credentials per service
Enable Multi-Factor Authentication (MFA)
Prefer SSH key-based authentication

4. Excessive Linux Capabilities (Privilege Escalation)Discovery:bash

getcap -r / 2>/dev/null
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

Exploitation:bash

python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

Root Cause:
Python binary retains unnecessary cap_setuid capability.Defense Recommendations:bash

sudo setcap -r /usr/bin/python3.8

Regularly audit capabilities: getcap -r /
Follow the principle of least privilege


Risk AssessmentVulnerability                          Severity
IDOR                                                  High
Sensitive Data Exposure (PCAP)                        High
Credential Reuse                                      Medium
Excessive Linux Capabilities                          Critical

Defense-in-Depth RecommendationsApplication LayerEnforce object-level authorization
Secure session management
Regular code reviews and access control testing

Network LayerEliminate plaintext protocols
Enforce encrypted communications
Monitor for suspicious traffic

System LayerRemove unnecessary capabilities
Apply least privilege principle
Regular auditing of privileged binaries

Lessons LearnedSmall issues like IDOR become critical when combined with poor credential hygiene and misconfigured capabilities.Key Takeaways:Strong authorization controls
Zero-trust + least privilege
Never transmit credentials in cleartext
Continuous security auditing

ReferencesOWASP Authorization Cheat Sheet
Linux Capabilities (man 7 capabilities)
Wireshark Documentation
Hack The Box - Cap Machine


Disclaimer
This analysis was performed in an authorized Hack The Box training environment for educational and defensive security purposes only.

Written: June 2026
Author: Security Researcher







