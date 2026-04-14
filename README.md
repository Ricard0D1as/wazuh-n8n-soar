markdown# Wazuh + n8n SOAR Lab

Automated SOC workflow: Wazuh FIM alerts → IP enrichment (AbuseIPDB + VirusTotal) → Discord notification using n8n.

Built as part of a Cybersecurity Specialization (CET) lab at CINEL, Lisbon.

---

## Objective

Demonstrate a real-world SOAR (Security Orchestration, Automation and Response) pipeline using open-source tools, triggered by File Integrity Monitoring (FIM) events in Wazuh.

---

## Architecture
Wazuh FIM (alert) → custom-n8n.py → nginx (HTTPS/443) → n8n webhook
→ AbuseIPDB enrichment
→ VirusTotal enrichment
→ Threat Filter (abuse score > 0)
→ Discord notification

---

## Stack

| Tool | Role |
|------|------|
| Wazuh 4.14.4 | SIEM / FIM / alert engine |
| n8n 2.16.0 | SOAR / workflow automation |
| nginx (alpine) | Reverse proxy + TLS termination |
| AbuseIPDB | IP reputation threat intelligence |
| VirusTotal | IP/file malware analysis |
| Discord | SOC notification channel |
| Docker | Container runtime |

---

## Repository Structure

    wazuh-n8n-soar/
    ├── README.md
    ├── docs/
    │   ├── 01-n8n-installation.md
    │   ├── 02-wazuh-integration.md
    │   └── 03-n8n-workflow.md
    ├── screenshots/
    │   ├── 01-installation/
    │   ├── 02-wazuh/
    │   ├── 03-n8n-workflow/
    │   └── 04-final-result/
    ├── workflow/
    │   └── wazuh-alert-enrichment.json
    └── config/
        └── wazuh-custom-integration.py

---

## Quick Start

### Prerequisites
- Wazuh Manager 4.x running
- Docker + Docker Compose
- AbuseIPDB API key (free tier)
- VirusTotal API key (free tier)
- Discord server with webhook

### 1. Deploy n8n
```bash
mkdir -p ~/n8n-docker/nginx/ssl
cd ~/n8n-docker

# Generate self-signed TLS certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/n8n.key \
  -out nginx/ssl/n8n.crt \
  -subj "/C=PT/ST=Lisboa/L=Lisboa/O=HomeLab/CN=YOUR_IP"

# Start containers
docker compose up -d
```

### 2. Deploy Wazuh integration
```bash
# Copy integration files
sudo cp config/custom-n8n /var/ossec/integrations/
sudo cp config/custom-n8n.py /var/ossec/integrations/

# Set permissions
sudo chmod 750 /var/ossec/integrations/custom-n8n*
sudo chown root:wazuh /var/ossec/integrations/custom-n8n*

# Restart Wazuh
sudo systemctl restart wazuh-manager
```

### 3. Import n8n workflow
- Open n8n at `https://YOUR_IP`
- Click **"..."** → **"Import from file"**
- Select `workflow/wazuh-alert-enrichment.json`
- Update API keys in AbuseIPDB and VirusTotal nodes
- Update Discord webhook URL
- Click **"Publish"**

---

## 🧪 Testing

Send a test alert with a known malicious IP:

```bash
curl -k -X POST https://YOUR_N8N_IP/webhook-test/wazuh-alert \
  -H "Content-Type: application/json" \
  -d '{
    "rule_id":"100201",
    "rule_desc":"File added to /root directory",
    "level":7,
    "src_ip":"185.220.101.45",
    "agent_name":"TestAgent",
    "timestamp":"2026-04-14T16:00:00"
  }'
```

Expected result in Discord:
Wazuh Security Alert
Rule: File added to /root directory
Source IP: 185.220.101.45
Abuse Score: 100%
VT Malicious Votes: 11
Agent: TestAgent

---

## 📸 Screenshots

### n8n Workflow Canvas
![Workflow Canvas](screenshots/03-n8n-workflow/false_branch_log.png)

### Discord Alert
![Discord Alert](screenshots/04-final-result/Discord_alert_msg.png)

---

## 📚 Documentation

- [01 — n8n Installation](docs/01-n8n-installation.md)
- [02 — Wazuh Integration](docs/02-wazuh-integration.md)
- [03 — n8n Workflow](docs/03-n8n-workflow.md)

---

## ⚠️ Security Notes

- Self-signed TLS certificate used — replace with a proper cert in production
- API keys stored as plain text in n8n nodes — use n8n credentials manager in production
- Wazuh integration uses `verify=False` for self-signed cert — acceptable in isolated lab

---

## 📄 License

MIT © Ricardo Dias
