# Wazuh SIEM & Endpoint Monitoring Lab

**Open-Source SIEM & XDR Security Platform — Deployment, Detection & Response Lab**

> Originally completed as a team project for *IT Elective 3 (Ethical Hacking)*. This repo documents my individual contribution and understanding of the deployment, architecture, and detection capabilities — reframed here as a standalone portfolio write-up.

## Project Summary

| Metric | Value |
|---|---|
| Tool | Wazuh |
| Version | 4.10.1 |
| Deployment Type | Docker (single-node) |
| Host OS | Debian 13, deployed as a VM on Proxmox VE |
| Active Agents | 4 (Windows + Linux/Kali endpoints) |
| Alerts Observed (24h) | 74 Medium / 0 High / 0 Critical |
| Framework | MITRE ATT&CK |

Wazuh is a free, open-source unified security platform combining **SIEM** (Security Information & Event Management), **EDR** (Endpoint Detection & Response), and **XDR** (Extended Detection & Response) capabilities. This lab covers the full lifecycle: provisioning infrastructure, deploying the platform, connecting endpoints, and running detection demonstrations across five core Wazuh modules.

## Why This Project Matters (for a security role)

This lab demonstrates the **defensive/blue-team side** of security work — a natural complement to the offensive vulnerability-assessment work in my [Nessus Vulnerability Assessment Lab](../Nessus-Vulnerability-Assessment-Lab). Together they show both halves of the security lifecycle: finding weaknesses (Nessus) and detecting exploitation of those weaknesses in real time (Wazuh).

Specifically, this project shows hands-on experience with:
- Standing up SIEM infrastructure from scratch (VM provisioning → OS install → Docker → Wazuh stack)
- Endpoint agent deployment and fleet management
- Reading and interpreting SIEM alerts, not just generating them
- Mapping detections to the **MITRE ATT&CK** framework — the language SOC teams use to communicate threat activity
- Identifying real indicators of compromise (a trojaned-binary/rootkit finding was caught live during this lab — see [Demo 2](docs/03-demonstration-scenarios.md#demo-2-malware-detection-via-rootcheck))

## Architecture

Wazuh follows a centralized, layered architecture: lightweight **Agents** on endpoints forward security data to a central server stack for processing, storage, and visualization. All inter-component communication is secured with TLS/SSL.

| Component | Role |
|---|---|
| **Wazuh Manager** | Analysis/orchestration engine. 3,000+ built-in detection rules, handles agent enrollment and Active Response. |
| **Wazuh Indexer** | Built on OpenSearch. Stores all normalized events/alerts; backend for all Dashboard queries. |
| **Wazuh Dashboard** | Web-based control center (OpenSearch Dashboards) for visualizations, alerting, and compliance reporting. |
| **Wazuh Agent** | Lightweight, multi-platform collector installed on endpoints (system logs, file changes, process activity, registry changes). Communicates over encrypted TCP 1514. |

## System Requirements Used

| Resource | This Deployment |
|---|---|
| CPU | 2 cores |
| RAM | 6 GB (6144 MB) |
| Storage | 50 GB |
| Host OS | Debian 13 (Trixie), CLI-only (SSH server + standard system utilities) |
| Virtualization | Proxmox VE |

## Repository Structure

```
.
├── README.md
└── docs/
    ├── 01-installation-and-deployment.md
    ├── 02-features-and-functionality.md
    ├── 03-demonstration-scenarios.md
    └── 04-advantages-limitations-conclusion.md
```

## References
- [Wazuh Official Documentation](https://documentation.wazuh.com/current/)
- [Wazuh Docker Deployment Guide](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html)
- [Wazuh Docker GitHub Repository](https://github.com/wazuh/wazuh-docker)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [NIST National Vulnerability Database](https://nvd.nist.gov)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)

## Disclaimer

All activity documented here was performed in an isolated lab environment on infrastructure the team owned/controlled, for academic and skill-demonstration purposes. Credentials, IP addresses, and personal contact details from the original coursework have been redacted or generalized for this public write-up.
