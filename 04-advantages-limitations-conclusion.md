# Advantages, Limitations & Conclusion

## Advantages

- **Completely free & open-source** — no licensing fees, full source-code access, actively maintained by Wazuh Inc. and a global community.
- **Unified platform** — combines SIEM, XDR, EDR, FIM, Vulnerability Detection, SCA, and compliance monitoring in one tool, replacing several disparate point solutions.
- **Highly scalable** — from single-node Docker deployments (as in this lab) up to distributed multi-node clusters handling millions of events/day.
- **Broad agent support** — Linux, Windows, macOS, Solaris, AIX, HP-UX.
- **Enterprise integrations** — AWS, Azure, GCP, Docker, Kubernetes, GitHub, Slack, PagerDuty, Jira, and more.
- **MITRE ATT&CK mapped** — detections speak the same language red and blue teams use industry-wide.
- **Active community** — millions of downloads, large GitHub presence, official Slack channel, optional paid enterprise support.
- **Regulatory compliance coverage** — built-in dashboards for PCI DSS, GDPR, HIPAA, and NIST 800-53.

## Limitations

- **Resource intensive at scale** — the Indexer (OpenSearch) needs significant RAM/CPU; large deployments require real capacity planning.
- **Complex advanced configuration** — custom decoders, rule sets, and Active Response scripts require deep familiarity with XML and Wazuh internals.
- **Learning curve** — the feature breadth is a lot to absorb for analysts new to SIEM platforms generally.
- **Docker networking complexity** — agent-to-manager communication can get fiddly in certain cloud/NAT setups.
- **Agent dependency** — full visibility requires an agent on every endpoint; agentless options are limited.
- **No default commercial support** — community support can be slower for complex issues; paid support is a separate subscription.

## Conclusion

This lab produced a fully functional Wazuh v4.10.1 deployment — Manager, Indexer, Dashboard, and connected Agents all operational — using Docker on a Debian Linux VM. With active agents connected and real alerts generated over 24 hours, the environment demonstrated Wazuh's core value: **unified security visibility across endpoints from a single dashboard.**

More importantly, this project surfaced a genuinely useful lesson in blue-team thinking: the SCA compliance failures (weak SSH hardening) directly explained *how* the rootkit finding in the rootcheck demo was likely possible in the first place. Seeing that cause-and-effect chain — misconfiguration → compromise → detection — is exactly the kind of correlation a SOC analyst needs to build the habit of making.

Understanding the **defensive perspective** — how a SIEM detects, logs, and correlates attack techniques — is essential knowledge on the offensive side too: it's what lets a penetration tester design more realistic engagements, understand detection coverage gaps, and deliver findings clients can actually act on.
