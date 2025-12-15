# 🚨 Incident Report: SSH Brute Force Simulation

**Date:** 2025-12-11
**Severity:** High (Level 10)
**Status:** Resolved (Simulation)
**Analyst:** [Your Name]

## 1. Executive Summary
An internal red-team simulation was conducted to test the detection capabilities of the SOC stack. A brute-force authentication attack was launched against an Azure Linux instance (`Azure-VM`) via the Tailscale overlay network. The SIEM (Wazuh) successfully correlated 40+ failed login attempts into a high-severity alert.

## 2. Infrastructure Details
* **Attacker IP:** `100.124.204.125` (Kali Linux / Tailscale)
* **Victim IP:** `100.92.151.36` (Azure-VM / Tailscale)
* **Target User:** `ankushc10`
* **Timeline:** Attack burst observed at `01:10:44` (Dec 11, 2025).

## 3. Threat Intelligence (MITRE ATT&CK)
The logs indicate a transition from simple guessing to high-volume brute force.

* **Tactic:** Credential Access (TA0006)
* **Technique 1:** Password Guessing (T1110.001) - *Individual login failures.*
* **Technique 2:** Brute Force (T1110) - *High frequency of failures.*

## 4. Detection & Evidence
The following alerts were generated in the Wazuh Dashboard:

| Rule ID | Level | Description | Count | Source |
| :--- | :--- | :--- | :--- | :--- |
| **5503** | 5 | **PAM: User login failed.** | 28 | `pam` |
| **5760** | 5 | **sshd: authentication failed.** | 42 | `sshd` |
| **2502** | **10** | **syslog: User missed the password more than one time.** | 7 | `syslog` |
| **40111** | **10** | **Multiple authentication failures.** | 2 | `syslog` |

### Log Analysis Sample
**Alert 5503 (Single Failure):**
```json
"full_log": "Dec 11 06:10:43 WazuhSIEM sshd[6079]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=100.124.204.125 user=ankushc10"
