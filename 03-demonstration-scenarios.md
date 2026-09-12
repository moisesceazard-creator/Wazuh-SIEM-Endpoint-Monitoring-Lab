# Demonstration Scenarios

> All demonstrations were conducted in a live Wazuh v4.10.1 lab environment. No external systems were involved — all activity was against team-owned, controlled endpoints for educational purposes only.

## Demo 1: Vulnerability Detection Scan

**Objective:** Show how Wazuh automatically discovers and classifies known CVEs present in installed software, with no manual scanning commands.

**Steps:**
1. Navigate to **Threat Intelligence → Vulnerability Detection**.
2. Select a connected agent to view its specific vulnerability findings.
3. Review the CVE list filtered by severity (Critical/High/Medium/Low).
4. Click a specific CVE to view its ID, CVSS score, affected package/version, and remediation.

**Results:** The scan surfaced a substantial number of unpatched CVEs on the Windows endpoint — 16 Critical and 410 High-severity findings across 600 scanned packages, indicating a significantly outdated environment with a wide exploitable attack surface.

**Analysis:** From an offensive perspective, Critical/High CVEs with public exploits would be the first targets during exploitation — they offer the highest success probability for the least effort. That Wazuh classified all of this automatically shows its value as a *continuous* vulnerability-management tool rather than a point-in-time scan.

## Demo 2: Malware Detection via Rootcheck

**Objective:** Demonstrate how Wazuh's rootcheck module automatically detects trojaned system binaries — a sign of active malware or post-exploitation activity — without manual scanning.

**Steps:**
1. Navigate to **Explore → Discover**.
2. Search `rule.groups: rootcheck`, time range: last 7 days.
3. Review results (60 hits, all from a single Linux agent).
4. Note the repeated finding: **"Trojaned version of file detected."**
5. Flagged files across all alerts: `/usr/bin/chsh`, `/bin/chsh`, `/usr/bin/chfn`, `/bin/chfn`, `/usr/bin/passwd`, `/bin/passwd`.
6. Expand an alert to review full forensic detail (file path, rule, signature used, timestamps).

**Results & Analysis:** These are core Linux utilities responsible for managing user credentials and shell access — exactly the binaries an attacker would replace after gaining root access, to establish persistence and harvest credentials. Trojaned versions appearing in **both** `/usr/bin` and `/bin`, combined with the rule firing repeatedly across multiple scan cycles, is consistent with a **rootkit installation** rather than an isolated incident. This was the most critical finding in the entire lab — the endpoint wasn't just vulnerable, it showed signs of actual system-level compromise.

## Demo 3: File Integrity Monitoring (FIM)

**Objective:** Demonstrate real-time detection of unauthorized modifications to critical system files, simulating post-compromise attacker behavior.

**Steps:**
1. Navigate to the **File Integrity Monitoring** module → Dashboard tab (active users/actions).
2. Check the Events chart for a modification spike.
3. Note files modified vs. added vs. deleted.
4. Inventory tab → browse monitored files.
5. Events tab → examine the alert and confirm the triggered rule.

**Results & Analysis:** FIM detected a modification to `/etc/hosts`, performed by the root user. A single file change looks minor in isolation, but edits to `/etc/hosts` are a known technique for **DNS hijacking** — redirecting legitimate domains to malicious IPs without touching DNS infrastructure itself. The alert captured the exact timestamp and responsible account. Even when an attacker changes things quietly, FIM leaves the forensic evidence a defender needs to investigate.

## Demo 4: Security Configuration Assessment (SCA)

**Objective:** Show Wazuh auditing endpoint configuration against CIS benchmarks and producing an actionable compliance score.

**Steps:**
1. Navigate to **Endpoint Security → Configuration Assessment**.
2. Select an active agent to view its SCA audit results.
3. Review the overall compliance score.
4. Expand check categories: description, expected value, actual value, result, remediation command.

**Results:** The audited Linux agent scored **18%** overall (3 passed / 13 failed / 7 not applicable) against the CIS Unix benchmark. Failed checks revealed a hardening-deficient configuration: root login over SSH permitted, public-key authentication disabled, password authentication enabled, empty passwords allowed, and no account-lockout policy.

**Analysis:** An SCA report like this functions as a pre-written attack-surface map — every failed check is a potential entry point or privilege-escalation vector. This 18% score directly explains *why* the rootkit finding in Demo 2 was possible on this same endpoint: the weak SSH/auth configuration was the likely initial access path.

## Demo 5: MITRE ATT&CK Framework View

**Objective:** Show how Wazuh contextualizes detected threats within MITRE ATT&CK, letting a security team understand the broader campaign and adversary behavior behind individual alerts.

**Steps:**
1. Navigate to **Threat Intelligence → MITRE ATT&CK**.
2. View the ATT&CK matrix heatmap of detected tactics/techniques.
3. Click a highlighted technique cell to see related alerts.
4. Trace a specific alert (e.g. brute-force activity) to its mapped technique.

**Results:** The Framework view highlighted detected techniques spanning the kill chain from **Initial Access through Impact**, distributed across multiple agents — indicating a broad range of suspicious behavior rather than one isolated incident. Different agents showed different dominant tactics (e.g. one agent driving Defense Evasion volume, another showing consistent Privilege Escalation activity).

**Analysis:** This view is the natural starting point for **incident-response prioritization** — identifying which agents need immediate attention and which tactics represent the most active threats at any given time, rather than triaging alerts one at a time with no shared context.
