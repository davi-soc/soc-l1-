# 🏠 ImoveisHub — SOC L1 Security Lab

> *"Today anyone can build a website with AI in a few hours."*  
> Let's see what happens after deploy.

**License:** MIT | **Lab:** Isolated | **MITRE ATT&CK**

---

## The Problem This Project Documents

A popular narrative has emerged in the tech and entrepreneurship world:

> *"You don't need developers. You don't need an IT team. Use Lovable for the frontend, Cursor for the code, ChatGPT for the content, Netlify to host — and your product is live."*

This narrative has one truth: **it is possible to ship a functional website in hours, without knowing how to code.**

What this narrative omits: **functional is not the same as secure.**

This project demonstrates, with real technical evidence, what happens when a system goes live without a security review — whether built by a non-technical founder with AI, an experienced developer with AI, or an entire team without an offensive security mindset.

---

## The Three Scenarios

### Scenario 1 — Non-Technical Founder + AI Stack

A business owner with no technical background uses Lovable, ChatGPT, and Cursor to build a real estate SaaS platform.

The site looks polished, works perfectly, and starts collecting real customer data: full names, CPF numbers, credit card digits, and account balances.

None of the AI tools stopped the process to say:

- *"You are storing CPF numbers in plain text."*
- *"Your login endpoint has no rate limiting."*
- *"Any user can manipulate tokens or bypass authentication."*

**Result:** A basic attacker compromises the system in minutes.

### Scenario 2 — Experienced Developer + AI Stack

A developer uses the same tools.

They fix obvious logic errors and improve the code structure — but offensive security is not part of the everyday development workflow.

Vulnerabilities like sensitive data exposure in APIs, weak JWT implementation, and missing access controls often ship undetected because they appear to work correctly from the developer's perspective.

**Result:** Slightly better. Still exploitable.

### Scenario 3 — Developer + AI + Security Process

Here the scenario truly changes.

Not because the developer is better, but because there is a process — someone thinking like an attacker before deploy, incorporating security-focused code review and threat hunting.

**Result:** Attack surface reduces dramatically.

---

## The Core Argument

**AI did not make software development insecure.**  
**Software development was already insecure.**

AI simply democratized access — now anyone can put a vulnerable system online, collect real user data, and not even know they are exposed.

This is exactly where the SOC analyst's work begins.

---

## What This Project Demonstrates

A fictional real estate SaaS platform (`ImoveisHub`) built with the stack that "gurus" recommend — professional visual, modern features, realistic customer data.

Underneath: **5 intentional vulnerabilities** that a SOC L1 analyst identifies, investigates, and documents using industry-standard tooling (**Splunk & Wireshark**).

---

## Lab Architecture

```text
┌────────────────────────────────────────────────────────────────┐
│              Isolated Network — 192.168.56.0/24                │
│                                                                │
│  VM1 — Debian Server          VM2 — Ubuntu Desktop              │
│  192.168.56.10                192.168.56.20                     │
│  ImoveisHub (Flask :5000)     Splunk + Logstash                 │
│                                                                │
│  VM3 — Kali Linux                                               │
│  192.168.56.30                                                   │
│  "the attacker"                                                 │
└────────────────────────────────────────────────────────────────┘
Documented Vulnerabilities
ID	Type	Endpoint / Tool	Severity	MITRE ATT&CK
VULN-1	Directory Fuzzing / Recon	ffuf / Multiple endpoints	Medium	T1595
VULN-2	Brute Force (Authentication)	POST /api/login (Status 401)	High	T1110.001
VULN-3	Successful Authentication / Session	POST /api/login (Status 200 OK)	Critical	T1078
VULN-4	Privilege Escalation via JWT	GET /api/usuarios	Critical	T1078 / T1548
VULN-5	Data Exfiltration in Mass	GET /api/usuarios (usuarios_dump.json)	Critical	T1041
Attack Chain & Investigation
PHASE 1 — Recon (Fuzzing)

ffuf maps hidden API endpoints, generating an influx of requests and status codes monitored via SIEM.

PHASE 2 — Brute Force

Continuous loop of POST /api/login triggering repeated 401 Unauthorized responses.

PHASE 3 — Access & Session

Successful credential match returning HTTP 200 OK along with the session token.

PHASE 4 — Privilege Escalation

Manipulation/foraging of token or authorized session state to access restricted administrative routes like /api/usuarios.

PHASE 5 — Exfiltration

Full data dump downloaded from the sensitive endpoint (usuarios_dump.json), captured via Wireshark (.pcap) and indexed in Splunk.

Repository Structure
soc-l1-lab/
├── README.md                     ← you are here
│
├── imoveis-hub/                  ← target application
│   ├── app.py                    ← Flask backend
│   ├── requirements.txt
│   ├── setup.sh                  ← automated Debian setup
│   └── deploy-frontend.sh        ← deploys Lovable build
│
├── attacks/                      ← Red Team write-ups
│   ├── 01-recon-ffuf.md
│   ├── 02-brute-force.md
│   ├── 03-login-success.md
│   ├── 04-privilege-escalation.md
│   └── 05-data-exfiltration.md
│
├── investigations/               ← Blue Team / Threat Hunting
│   └── splunk-queries.spl
│
└── screenshots/                  ← evidence per phase
Lab Setup
Requirements
VirtualBox with Host-Only network 192.168.56.0/24
VM1: Debian 12 (Target Application)
VM2: Ubuntu 22.04 Desktop with Splunk Enterprise Free + Logstash (SIEM)
VM3: Kali Linux (Attacker)
VM1 — Debian (Target Application)
Clone and Run Automated Setup
git clone https://github.com/your-username/soc-l1-lab.git
cd soc-l1-lab/imoveis-hub
sudo bash setup.sh
Deploy the Lovable Frontend Build
scp lovable-build.zip nexus@192.168.56.10:~
sudo bash deploy-frontend.sh ~/lovable-build.zip

Access the application at:

http://192.168.56.10:5000
Test Accounts
E-mail	Senha	Role
admin@imoveis.com	admin123	Administrator
joao@imoveis.com	senha123	Broker
maria@imoveis.com	maria2024	Broker

⚠️ These credentials are intentionally exposed because this is an isolated educational lab.

Useful Splunk Queries (SIEM)
Phase 1 — Fuzzing / Recon
index="linux" earliest="-24h"
| stats count as Total_Tentativas by clientip, uri_path, status
| sort - Total_Tentativas
Phase 2 — Brute Force (401s)
index="linux" "/api/login" status="401"
| stats count as Total_Falhas by clientip, host
| where Total_Falhas >= 10
| sort - Total_Falhas
Phase 4 & 5 — Sensitive Endpoint Access / Exfiltration
index="linux" uri_path="/api/usuarios"
| stats count by clientip, method, uri_path, status, useragent
| sort - count
Useful Commands (Debian Server)
Check Service Status
systemctl status imoveis-hub
Follow Application Logs in Real Time
tail -f /var/log/imoveis-hub/access.log
Restart Application
systemctl restart imoveis-hub
Verify Port 5000 Is Listening
ss -tlnp | grep 5000
Legal Disclaimer

This project intentionally contains security vulnerabilities for strictly educational purposes in an isolated lab environment.

Never deploy this application on public networks, production systems, cloud environments, or any infrastructure accessible from the internet.

The authors assume no responsibility for misuse of this software outside controlled lab environments.

License

This project is licensed under the MIT License.

MITRE ATT&CK Coverage
Technique	ID	Phase
Active Scanning	T1595	Recon
Brute Force	T1110.001	Authentication
Valid Accounts	T1078	Initial Access
Abuse Elevation Control Mechanism	T1548	Privilege Escalation
Exfiltration Over C2 Channel	T1041	Data Exfiltration
