# SOC-Automation
A simple n8n automation for automated detection of attacks ,reporting and response. This project utilizes the tools like suricata for rule based detection and send alert through splunk which triggers the automation . 

Overview
This project implements an automated cybersecurity pipeline for log analysis, threat detection, reporting, and incident response. It integrates OSSEC for host-based intrusion detection, Suricata for network monitoring, Splunk for centralized log management, n8n for workflow orchestration, Jira for ticketing, Perplexity AI nodes for intelligent analysis, and Gmail nodes for notifications.

The system collects logs from OSSEC and Suricata, processes them in Splunk for correlation, uses n8n to trigger AI-driven analysis via Perplexity, creates Jira tickets for high-severity alerts, and sends summaries via Gmail.

Designed for virtualized environments like Kali Linux, Windows Server, and Docker containers, aligning with your cybersecurity lab setups.

Architecture
The pipeline follows this flow:

Data Collection: OSSEC monitors host logs and file integrity; Suricata captures network events (e.g., brute-force, DoS simulations).

Ingestion: Logs forward to Splunk indexers via syslog or agents.

Analysis: n8n workflows query Splunk, invoke Perplexity for anomaly detection (e.g., extracting errors by context ID), and score threats.

Response: High-risk events create Jira issues; Gmail notifies your team; optional auto-remediation scripts.

Key integrations:

n8n triggers on Splunk alerts or schedules.

Perplexity nodes parse logs into tables (e.g., Error Type | Related Logs).

Jira nodes auto-assign tickets with log excerpts.

Gmail for executive summaries.

Components
Tool	Role	Configuration Highlights
OSSEC	Host IDS, log monitoring	Custom rules for your DVWA/Hydra tests; active response enabled.
Suricata	Network IDS	Rulesets for exploits; eve.json output to Splunk.
Splunk	SIEM, querying	Indexes for ossec/suricata; SPL for correlation searches.
n8n	Orchestration	Workflows with cron triggers, HTTP to Splunk/Jira.
Perplexity	AI log analysis	Prompts for error extraction, threat classification.
Jira	Incident ticketing	API integration for auto-issue creation.
Gmail	Notifications	Send analyzed reports, attach Splunk dashboards.
Setup Instructions
Deploy VMs: Kali (Suricata/OSSEC), Ubuntu (n8n/Splunk forwarder), Windows Server (AD simulation).

Install tools:

OSSEC: yum install ossec-hids-server; add Suricata log parser.

Suricata: suricata -c /etc/suricata/suricata.yaml.

n8n: Docker (docker run -p 5678:5678 n8nio/n8n); add credentials for Splunk/Jira/Gmail/Perplexity.

Configure Splunk: Add inputs for OSSEC/Suricata; create alerts feeding n8n webhooks.

Build n8n workflows: Import JSON from your past chats (e.g., log fetch → Perplexity → Switch node → Jira/Gmail).

Test: Simulate attacks (nmap, hydra); verify end-to-end alerts.

Workflows Example
Daily Log Scan: Cron → Splunk query → Perplexity analysis → Gmail report.

Alert Response: Splunk webhook → n8n → Threat score > 7? → Jira ticket + Gmail.

Example n8n prompt for Perplexity:

text
Analyze logs for errors by context ID. Output table:
Error Type | Related Logs
Usage and Testing
Run in your VirtualBox/WSL2 lab. Monitor via Splunk dashboard. Scale with Docker Compose for production-like testing.

Past chats from ~Nov 2025 detailed your n8n JSON exports and Splunk queries—import those for quick start.

Future Enhancements
Add ML models (e.g., transformers for anomaly detection).

Integrate blockchain for log integrity.

Expand to Spark for big data logs.
