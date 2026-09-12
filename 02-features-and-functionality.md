# Features & Functionality

Wazuh organizes its capabilities into four security domains: **Endpoint Security, Threat Intelligence, Security Operations, and Cloud Security.** Below are the modules used in this lab, what they do, and why they matter from a security-analyst perspective.

## Vulnerability Detection

Automated identification of known CVEs across all monitored endpoints, without any manual scan commands.

- The **Wazuh Agent** collects a complete software inventory (installed packages, versions, paths).
- The **Wazuh Manager** correlates this inventory against the National Vulnerability Database (NVD), Red Hat Security Database, Debian/Ubuntu CVE feeds, and vendor advisories.
- Matches are classified by severity (Critical/High/Medium/Low) using CVSS score.
- Each finding includes: CVE ID, CVSS score, affected package, version, and recommended remediation.

**Why it matters:** this is continuous vulnerability management, not a periodic-scan snapshot — the Dashboard reflects endpoint exposure in near real time.

## File Integrity Monitoring (FIM)

Continuously watches specified files/directories for unauthorized changes, generating cryptographic checksums (MD5/SHA1/SHA256) and alerting on any modification.

- Detects file creation, modification, and deletion in real time.
- Monitors permission, ownership, and attribute changes.
- Records before/after checksums for forensic comparison.
- Configurable scope (system files, config directories, web roots, custom paths).
- Works on Windows (including registry monitoring) and Linux/macOS.

**Why it matters:** modifications to sensitive files like `/etc/hosts` are a known technique for DNS hijacking — FIM provides the forensic trail even when an attacker moves quietly.

## Log Data Analysis

A high-performance, rule-based engine that continuously collects, normalizes, parses, and analyzes logs from hundreds of sources in real time, using 3,000+ built-in rules to detect known attack signatures, anomalous patterns, and policy violations.

## Security Configuration Assessment (SCA)

Automatically audits endpoint configuration against industry benchmarks (e.g. CIS), identifying misconfigurations such as weak SSH settings, open services, disabled firewalls, and insecure file permissions — then produces a compliance score with actionable remediation guidance.

## Active Response

Automated, rule-triggered response actions (e.g. blocking an IP, disabling an account) that the Manager can execute when specific alert conditions are met — turning detection into containment without waiting on a human in the loop.

## MITRE ATT&CK Integration

All detected alerts are mapped to the **MITRE ATT&CK framework** — the industry-standard knowledge base of adversary tactics, techniques, and procedures (TTPs). This enriches every alert with strategic context: not just *what* happened, but *what the attacker was trying to achieve*.
