# ImoveisHub — SOC L1 Security Lab

A fictional real estate platform designed as a target application for cybersecurity training labs.

The application mimics a real-world business platform while intentionally containing **five security vulnerabilities** that can be identified, investigated, and exploited in a controlled educational environment.

> ⚠️ This project is intended exclusively for isolated laboratory environments and security training purposes.

---

## Features

* Flask-based backend API
* Static frontend deployment support
* JWT authentication
* Systemd service integration
* Automated Debian setup script
* Simulated real-world business workflows
* Intentionally vulnerable endpoints for SOC and security analyst training

---

## Repository Structure

```text
imoveis-hub/
├── app.py                # Flask backend (API + frontend serving)
├── requirements.txt      # Python dependencies
├── setup.sh              # One-command Debian installation
├── deploy-frontend.sh    # Deploys the Lovable frontend build
└── dist/                 # Frontend build directory (created during setup)
```

---

## Prerequisites

* Debian-based Linux distribution
* Root or sudo privileges
* Internet access for package installation
* Frontend build exported from Lovable

---

## Installation Guide

### Step 1 — Clone the Repository

```bash
# On VM1 (192.168.56.10)
git clone https://github.com/your-username/soc-l1-lab.git
cd soc-l1-lab/imoveis-hub
```

### Step 2 — Run the Setup Script

```bash
sudo bash setup.sh
```

The script will automatically:

* Update system packages
* Install Python, pip, and UFW
* Create the non-root user `nexus`
* Create a Python virtual environment
* Install application dependencies
* Register the application as a systemd service
* Configure the firewall (ports 22 and 5000)

---

### Step 3 — Export the Frontend from Lovable

From Lovable:

1. Click **Share**
2. Select **Export**
3. Choose **Download as ZIP**

Transfer the ZIP file to the target VM:

```bash
scp lovable-build.zip nexus@192.168.56.10:~
```

---

### Step 4 — Deploy the Frontend

```bash
sudo bash deploy-frontend.sh ~/lovable-build.zip
```

Once completed, access the application:

```text
http://192.168.56.10:5000
```

---

## Test Accounts

| Email                                         | Password  | Role          |
| --------------------------------------------- | --------- | ------------- |
| [admin@imoveis.com](mailto:admin@imoveis.com) | admin123  | Administrator |
| [joao@imoveis.com](mailto:joao@imoveis.com)   | senha123  | Broker        |
| [maria@imoveis.com](mailto:maria@imoveis.com) | maria2024 | Broker        |

---

## Deliberate Vulnerabilities

| ID     | Vulnerability                           | Endpoint               | Severity |
| ------ | --------------------------------------- | ---------------------- | -------- |
| VULN-1 | SQL Injection                           | POST /api/login        | Critical |
| VULN-2 | Unrestricted Brute Force                | POST /api/login        | High     |
| VULN-3 | Insecure Direct Object Reference (IDOR) | GET /api/clientes/{id} | Critical |
| VULN-4 | Sensitive Data Exposure                 | GET /api/usuarios/me   | Critical |
| VULN-5 | Weak JWT Secret                         | Authentication System  | High     |

---

## Useful Debian Commands

### Check service status

```bash
systemctl status imoveis-hub
```

### View application logs

```bash
tail -f /var/log/imoveis-hub/access.log
```

### Restart the application

```bash
systemctl restart imoveis-hub
```

### Verify port 5000 is listening

```bash
ss -tlnp | grep 5000
```

---

## Suggested Lab Architecture

```text
SOC Analyst VM
      │
      ▼
┌──────────────────┐
│   ImoveisHub     │
│ Vulnerable App   │
└──────────────────┘
      │
      ▼
Filebeat / Syslog
      │
      ▼
    Splunk
```

---

## Disclaimer

This project intentionally contains security vulnerabilities for educational purposes.

Do not deploy this application on public networks, production systems, cloud environments, or any infrastructure accessible from the Internet.

The authors assume no responsibility for misuse of this software outside controlled laboratory environments.
