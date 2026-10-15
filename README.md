# Autonomous SOC Incident Response & Threat Detection Pipeline (n8n + Multi-AI Agent)

An automated Security Operations Center (SOC) incident response and threat detection pipeline built on **n8n**. This workflow captures real-time network and host alerts triggered by attack simulations (**Hydra** and **Slowloris**) using **Suricata (NIDS)** and **OSSEC (HIDS)**. It then leverages three specialized AI Agents (**Triage**, **Response**, and **Reporting**) operating in sequence to autonomously investigate SIEM events, execute active network/directory containment, and report actionable threat intelligence.

---

## 🏗️ Architecture & Workflow Overview

                  ┌─────────────────────────────────┐
                  │  Simulated Attacks              │
                  │  - Hydra (Brute-Force)          │
                  │  - Slowloris (DoS)              │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                  ┌─────────────────────────────────┐
                  │  Detection Layer                │
                  │  - Suricata (NIDS)              │
                  │  - OSSEC (HIDS)                 │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                  ┌─────────────────────────────────┐
                  │  Splunk SIEM Aggregation        │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                  ┌─────────────────────────────────┐
                  │    1. Triage AI Agent           │
                  │  - Query Splunk (SIEM)          │
                  │  - HTTP Threat Intel (VT)       │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                  ┌─────────────────────────────────┐
                  │    2. Response AI Agent         │
                  │  - Active Directory (LDAP)      │
                  │  - Linux iptables (SSH)         │
                  └────────────────┬────────────────┘
                                   │
                                   ▼
                  ┌─────────────────────────────────┐
                  │    3. Reporting AI Agent        │
                  │  - Jira Ticket Creation         │
                  │  - Slack Channel Alerting       │
                  └─────────────────────────────────┘

The pipeline decomposes complex SOC threat detection and incident handling into three autonomous agents:

### 1. Detection & Simulation Layer
* **Attack Simulation:**
  * **Hydra:** Simulates high-frequency credential brute-force attacks against target network services (SSH, FTP, Web forms).
  * **Slowloris:** Executes Layer 7 Denial of Service (DoS) attacks by keeping HTTP connections open asynchronously.
* **Network & Host Detection:**
  * **Suricata (NIDS):** Inspects raw network packets to identify malicious signature patterns, flood attacks, and port scans.
  * **OSSEC (HIDS):** Performs real-time host-level log analysis, file integrity monitoring (FIM), and failed login correlation.

### 2. Triage Agent (Investigation & Threat Intelligence)
* **Splunk Integration:** Dynamically forms and executes Search Processing Language (SPL) queries against Splunk SIEM to extract context from ingested Suricata and OSSEC logs (Event IDs, IP addresses, user accounts, target services).
* **HTTP Enrichment Tool:** Queries external threat intelligence APIs (e.g., VirusTotal, AbuseIPDB) to check IP reputations and gather threat scores.
* **Analysis:** Aggregates raw logs and external intelligence to evaluate the incident's severity and malicious confidence score.

### 3. Response Agent (Active Containment)
* **LDAP Node (Active Directory):** Disables compromised domain user accounts, revokes active privileges, or moves targeted objects to quarantined Organizational Units (OUs).
* **SSH Node (Network Firewall Enforcement):** Connects to target Linux hosts or firewall gateways via SSH to dynamically insert `iptables` drop rules, blocking malicious IP addresses at the host/network level (e.g., mitigating active Hydra brute-force IPs or Slowloris connections).

### 4. Reporting Agent (Communication & Ticket Lifecycle)
* **Jira Node:** Automatically creates structured incident tickets containing triage findings, executed containment actions, IOC lists, and initial timeline details.
* **Slack Node:** Formats rich markdown alerts and posts high-priority incident notifications to designated SOC channels (`#soc-alerts`) for human analyst visibility.

---

## 🛠️ Prerequisites & Connected Integrations

| Tool / Layer | Component Category | Function in Pipeline |
| :--- | :--- | :--- |
| **Hydra** | Attack Simulation | Executes credential brute-force testing against target services. |
| **Slowloris** | Attack Simulation | Simulates Layer 7 HTTP Denial-of-Service (DoS) connection flooding. |
| **Suricata** | NIDS | Deep packet inspection for network-based anomaly and attack detection. |
| **OSSEC** | HIDS | Host log analysis, rootkit detection, and file integrity monitoring. |
| **Splunk Enterprise** | SIEM Ingestion | Aggregates logs from Suricata, OSSEC, and system event logs. |
| **n8n** | SOAR / AI Agent | Multi-agent workflow orchestration and automated response execution. |
| **VirusTotal / Threat APIs** | HTTP / REST | IOC Reputation Scoring & Threat Intelligence Enrichment. |
| **Active Directory** | LDAP / LDAPS | Account Containment & User Status Modification. |
| **Linux Endpoint / Gateway** | SSH / iptables | Dynamic Host Firewall Rule Insertion & Connection Dropping. |
| **Jira Software** | REST API | Incident Tracking & Ticket Lifecycle Management. |
| **Slack** | Webhook / API | Real-Time Team Notifications and Incident Alerts. |

---

## 🚀 Setup & Installation

### 1. Import Workflow into n8n
1. Clone this repository:
   ```bash
   git clone [https://github.com/your-username/soc-automation-n8n.git](https://github.com/your-username/soc-automation-n8n.git)
In your n8n web interface, go to Workflows $\rightarrow$ Import from File.Select the provided workflow.json file from this repository.2. Configure Credentials in n8nEnsure the following credentials are setup under your n8n environment:Splunk API: Server URL, API token, or username/password.HTTP Request Header Auth (VirusTotal): API Key passed as x-apikey.LDAP Credentials: Bind DN, password, and LDAPS host connection.SSH Credentials: Host IP, Port (22), Username, and Private Key / Password.Jira & Slack OAuth / API Tokens: Authorized workspace API tokens.🛡️ Security Considerations & Safety ControlsLeast-Privilege SSH Access: The SSH credential utilized by n8n should be restricted using sudoers rules to only execute specific iptables commands rather than granting full root shell access:Bash# Example sudoers entry for n8n SSH user
n8n_user ALL=(ALL) NOPASSWD: /sbin/iptables
Controlled Attack Simulation: Always ensure Hydra and Slowloris scripts are executed exclusively within authorized lab environments or isolated subnet segments.Human-in-the-Loop Option: A manual approval node can be inserted between the Triage Agent and Response Agent to require analyst authorization prior to executing firewall or directory containment.📄 LicenseThis project is open-source and available under the MIT License.
