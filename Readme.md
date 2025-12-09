# 🛡️ Hybrid SOC Automation Lab

## Project Overview
This project involves building an enterprise-grade Security Operations Center (SOC) from scratch to monitor hybrid infrastructure. The goal is to simulate real-world attack lifecycles (Red Team) and develop detection engineering skills (Blue Team) using open-source tools.

**Current Status:** Phase 1 (Infrastructure & Connectivity) Complete.

## 🏗️ Architecture (Phase 1)
* **SIEM Core:** Wazuh Manager running in Docker (on Linux Host).
* **Endpoints:** Windows 11 Enterprise (Surface monitoring).
* **Networking:** Zero-Trust Mesh VPN via **Tailscale** to bypass CGNAT and connect disparate networks.
* **Telemetry:** Sysmon (Windows) + Auditd (Linux).

## 🔧 Engineering Challenges & Solutions
During the initial deployment, several infrastructure hurdles were resolved:

### 1. Docker Network Isolation vs. Host Firewall
* **Problem:** The Wazuh Manager (Docker) was running, but external agents could not connect.
* **Root Cause:** The Linux Host Firewall (UFW) was blocking traffic on ports 1514/1515, preventing it from reaching the Docker bridge interface.
* **Solution:** Implemented UFW rules to explicitly allow traffic on the Tailscale interface (`tailscale0`).

### 2. Windows Service Configuration & Race Conditions
* **Problem:** The Wazuh Agent service on Windows failed to start during automated PowerShell deployment.
* **Root Cause:** A race condition where the service start command triggered before the MSI installer finished.
* **Solution:** Refactored the deployment script to include `Start-Process -Wait` and manual configuration validation.

### 3. Syntax Sensitivity in Config Injection
* **Problem:** Agent authentication failed with `Invalid agent name` errors.
* **Analysis:** PowerShell injected single quotes (`'`) into the `ossec.conf` XML values (e.g., `<address>'100.x.x.x'</address>`), which the parser rejected.
* **Fix:** Manually sanitized the XML configuration to enforce strict string formatting.

## 🚀 Next Steps
* [ ] Deploy Sysmon with SwiftOnSecurity configuration.
* [ ] Execute Atomic Red Team credential dumping attacks (T1003).
* [ ] Write custom Wazuh detection rules for "Impossible Traveler" scenarios.
