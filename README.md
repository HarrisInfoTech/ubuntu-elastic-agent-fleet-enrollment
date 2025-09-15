# 🐧 Ubuntu 24.04 SSH Server + Elastic Agent (Fleet) — Vultr

I deployed an **Ubuntu 24.04** VM on **Vultr**, SSH’d in with PowerShell, updated packages, created a **Linux agent policy** in Kibana, and **enrolled the Elastic Agent**. I hit a self-signed certificate error during enrollment, re-ran with `--insecure` for the lab, and verified the host in **Discover** (I saw **three hosts**, including this Linux box).

> 🔐 Screenshot note: blur/replace real IPs/hostnames. Use placeholders like `203.0.113.25` (public) and `10.0.0.x` (private).

---

## 📌 Project Overview
- **Cloud:** Vultr (Cloud Compute)
- **OS:** Ubuntu Server **24.04** (1 CPU / 1 GB RAM)
- **Access:** SSH (PowerShell → `ssh root@<ip>`)
- **Elastic:** Fleet already set up from prior lab (Fleet Server + Kibana)
- **Goal:** Enroll a Linux host into Fleet under **Linux-policy**; confirm data in Discover

---

## 🧱 Lab Architecture

| Component                     | Role / Purpose                              | Notes / Ports |
|------------------------------|---------------------------------------------|---------------|
| Ubuntu 24.04 (this VM)       | Elastic Agent (enrolled)                    | Outbound → Fleet Server `:8220` |
| Fleet Server                 | Agent mgmt & policy                         | `:8220/tcp` (restricted to agent sources) |
| Kibana                       | UI to create **Linux-policy** & Add Agent   | `:5601/tcp` (my IP only) |
| Elasticsearch                | Storage / search                             | `:9200/tcp` (private or allowlisted) |

📸 *Diagram placeholder*  
![Architecture](./screenshots/elastic-linux-architecture.png)

---

## ✅ Prerequisites
- Vultr account + ability to open/limit firewall rules
- Existing Elastic Stack with **Fleet Server** and **Kibana** reachable from the agent
- Enrollment token for the policy you’ll use (created in Kibana)

---

## 🚀 Steps (What I Actually Did)

### 1) Deploy the server on Vultr
- **Cloud Compute CPU**, **Ubuntu 24.04**, **1 CPU / 1 GB RAM**
- Give it a **label/hostname**, **Deploy**

📸 `vultr-deploy-ubuntu-24.png`

---

### 2) SSH in (PowerShell) and update the box
```powershell
# From my PC (PowerShell)
ssh root@<public-ip>
bash
Copy code
# On the Ubuntu server
apt-get update && apt-get upgrade -y
(Optional hardening after the lab: create a sudo user, disable root SSH, enforce key auth.)

📸 powershell-ssh-root.png, apt-upgrade.png

3) Create a Linux policy in Kibana
Management → Fleet → Agent policies → Create policy

Name: Linux-policy → Create

📸 kibana-create-linux-policy.png

Note: If you see no data after enrollment, add the System integration to Linux-policy.

4) Add the Linux agent (Fleet → Add agent)
Fleet → Agents → Add agent

Where to add this agent: select Linux-policy

Enrollment: choose Linux x86_64 and Copy the install command

📸 fleet-add-agent-linux.png, fleet-copy-linux-command.png

5) Run the enrollment command on the server
Back on your SSH session, paste the command you copied (example shape below):

bash
Copy code
# Example (yours will include your Fleet URL + token)
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-<VERSION>-linux-x86_64.tar.gz
tar xzf elastic-agent-<VERSION>-linux-x86_64.tar.gz
cd elastic-agent-<VERSION>-linux-x86_64
sudo ./elastic-agent install \
  --url=https://<fleet-server-host>:8220 \
  --enrollment-token=<YOUR_ENROLLMENT_TOKEN>
I initially got a self-signed certificate error, so I re-ran with --insecure (lab only):

bash
Copy code
sudo ./elastic-agent install \
  --url=https://<fleet-server-host>:8220 \
  --enrollment-token=<YOUR_ENROLLMENT_TOKEN> \
  --insecure
📸 agent-enroll-error-self-signed.png, agent-enroll-with-insecure.png

⚠️ Production note: Prefer installing your Fleet Server CA and using --certificate-authorities=/path/ca.crt instead of --insecure.

6) Confirm agent online in Fleet
Fleet → Agents shows the new Linux host as Healthy/Online

Policy = Linux-policy

📸 fleet-agent-online-linux.png

🏁 Results

Ubuntu 24.04 VM reachable via SSH

Linux-policy created and Elastic Agent enrolled (Fleet)

Worked around self-signed cert with --insecure for the lab

Verified in Kibana Discover: I saw three hosts, including the new Linux machine
