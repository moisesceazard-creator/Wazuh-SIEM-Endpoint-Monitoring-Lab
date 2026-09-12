# Installation & Deployment

## Deployment Overview

Wazuh v4.10.1 was deployed using the official **Docker single-node configuration**, which packages the Manager, Indexer, and Dashboard as a unified Docker Compose stack. This reduces installation complexity while keeping full platform functionality. The host is a Debian Linux VM provisioned on **Proxmox VE**, accessible remotely via **Tailscale** (a zero-config VPN mesh) for team collaboration.

## Part 1 — Provisioning the VM (Proxmox)

1. In the Proxmox web interface, select the target node and click **Create VM**.
2. Assign a unique VM ID and a descriptive name.
3. **OS:** Debian 13.1.0 (`debian-13.1.0-amd64-netinst.iso`), Guest OS type Linux, kernel 6.x.
4. **System:** default settings (SeaBIOS, VirtIO SCSI single controller).
5. **Disk:** 50 GB.
6. **CPU:** 1 socket / 2 cores.
7. **Memory:** 6144 MiB (6 GB).
8. **Network:** VirtIO (paravirtualized) model, firewall enabled.
9. Review the configuration on the Confirm tab and finish — the VM appears in the resource tree.

## Part 2 — OS Installation (Debian 13)

1. Boot the VM and choose **Graphical Install**.
2. Language: English. Location/keyboard layout: set to match your region.
3. Configure hostname and domain name for the system.
4. Set the root password and create a standard (non-root) user account for day-to-day access — **use strong, unique passwords**, not shared or reused ones.
5. **Disk partitioning:** Guided – use entire disk → all files in one partition (recommended for new users/lab setups) → finish partitioning → confirm to write changes.
6. Skip scanning additional install media.
7. **Package mirror:** choose a mirror geographically close to you.
8. Decline the popularity-contest survey (optional telemetry).
9. **Software selection:** check only **SSH server** and **standard system utilities** — no desktop environment needed for a headless server.
10. Install the GRUB bootloader to the primary disk (e.g. `/dev/sda`).
11. Reboot into the new system once installation completes.

## Part 3 — Remote Access Setup (Tailscale)

To allow the whole team to reach the VM without exposing it to the public internet, [Tailscale](https://tailscale.com/) was installed on the Debian host and the machine was shared with teammates via the Tailscale admin console (`https://login.tailscale.com/admin/machines`). Each teammate accepts a device-share invite, after which the shared machine appears in their tailnet, tagged **"Shared In"**.

> **Security note:** Only enable read/connect permissions you actually need when sharing a device — Tailscale's sharing model lets you grant "can connect to this device" without granting broader network access.

## Part 4 — Installing Docker & Deploying Wazuh

**Step 1 — Update the system**
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

**Step 2 — Install Docker Engine & Docker Compose**
```bash
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor \
  -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

**Step 3 — Increase virtual memory limit** (required by the Wazuh Indexer / OpenSearch)
```bash
# Apply immediately
sudo sysctl -w vm.max_map_count=262144

# Apply permanently (survives reboot)
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

**Step 4 — Clone the official Wazuh Docker repository**
```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.10.1
cd wazuh-docker/single-node
```

**Step 5 — Generate TLS/SSL certificates** for secure internal communication between Manager, Indexer, and Dashboard
```bash
docker compose -f generate-indexer-certs.yml run --rm generator
# Certificates written to: ./config/wazuh_indexer_ssl_certs/
```

**Step 6 — Deploy the full stack**
```bash
docker compose up -d
```
Docker pulls and starts all three services:
```
Pulling wazuh/wazuh-manager:4.10.1   ... done
Pulling wazuh/wazuh-indexer:4.10.1   ... done
Pulling wazuh/wazuh-dashboard:4.10.1 ... done
Creating single-node-wazuh.manager-1   ... done
Creating single-node-wazuh.indexer-1   ... done
Creating single-node-wazuh.dashboard-1 ... done
```

**Step 7 — Verify container health**
```bash
docker compose ps
```
```
NAME                            IMAGE                          STATUS
single-node-wazuh.manager-1     wazuh/wazuh-manager:4.10.1     Up (healthy)
single-node-wazuh.indexer-1     wazuh/wazuh-indexer:4.10.1     Up (healthy)
single-node-wazuh.dashboard-1   wazuh/wazuh-dashboard:4.10.1   Up (healthy)
```

**Step 8 — Access the Dashboard**

Navigate to `https://<vm-ip-or-tailscale-address>`, accept the self-signed certificate warning, and log in with the default `admin` credentials.

> ⚠️ **Change the default admin password immediately after first login.** Ship a fresh, unique password rather than leaving Wazuh's documented default in place — this is the single most common misconfiguration on internet-facing Wazuh instances.

## Part 5 — Agent Deployment

Agents were deployed to endpoints (including a Kali Linux box) directly from the Dashboard's guided wizard:

1. In the Dashboard, go to **Agents** → **Deploy new agent**.
2. Select the target OS package (Linux/Windows/macOS).
3. Enter the **Manager's server address** (the Wazuh host's IP).
4. Assign a unique agent name.
5. Copy the generated install command and run it on the target machine as root:
   ```bash
   wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.10.1-1_amd64.deb \
     && sudo WAZUH_MANAGER='<manager-ip>' WAZUH_AGENT_NAME='<agent-name>' dpkg -i ./wazuh-agent_4.10.1-1_amd64.deb
   ```
6. Start and enable the agent:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```
7. The agent auto-registers with the Manager and appears as **Active** under the Agents Summary.

## Dashboard Overview

The Wazuh Dashboard is the central command interface for the whole platform, providing real-time visibility into security events, agent health, threat intelligence, and regulatory compliance — Endpoint Security, Threat Intelligence, Security Operations, and Cloud Security modules all in one pane.
