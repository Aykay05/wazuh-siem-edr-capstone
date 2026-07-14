# Wazuh SIEM/EDR Capstone Project

Cloud-native SIEM/EDR detection engineering lab built on Wazuh 4.9.2, deployed on AWS EC2 with multi-OS agent monitoring, live attack simulations via Atomic Red Team, and Sysmon integration.

---

## Architecture
┌─────────────────────────────────────────────────┐
│              AWS (us-east-1)                     │
│                                                  │
│  ┌─────────────────┐   ┌─────────────────────┐  │
│  │ ubuntu-agent-01 │   │  windows-agent-01   │  │
│  │ Ubuntu 22.04    │   │  Windows Server 2022│  │
│  │ t3.micro        │   │  c7i-flex.large     │  │
│  │ [Wazuh Agent]   │   │  [Wazuh Agent]      │  │
│  │ [Auditd]        │   │  [Sysmon v15]       │  │
│  └────────┬────────┘   └──────────┬──────────┘  │
│           └──────────┬────────────┘             │
│                      │ TCP 1514/1515            │
│           ┌──────────▼──────────┐               │
│           │   wazuh-manager     │               │
│           │   t3.small          │               │
│           │   Wazuh 4.9.2       │               │
│           │   + Indexer         │               │
│           │   + Dashboard       │               │
│           └─────────────────────┘               │
└─────────────────────────────────────────────────┘

## Technologies Used

| Category | Tools |
|---|---|
| Cloud | AWS EC2, Security Groups, VPC |
| SIEM/EDR | Wazuh 4.9.2 (Manager + Indexer + Dashboard) |
| Endpoints | Ubuntu 22.04 LTS, Windows Server 2022 |
| Threat Simulation | Atomic Red Team, PowerShell 7 |
| Windows Telemetry | Sysmon v15 (SwiftOnSecurity config) |
| Detection | Custom Sigma rules, Wazuh XML rules |
| Framework | MITRE ATT&CK |

---

## Attack Simulations

| TTP | Technique | Platform | Result |
|---|---|---|---|
| T1059.001 | PowerShell / Mimikatz | Windows | ✅ Detected |
| T1087.002 | Account Discovery | Windows | ✅ Detected |
| T1003.001 | LSASS Credential Dump | Windows | ✅ Detected |
| T1082 | System Information Discovery | Windows | ✅ Detected |
| T1049 | Network Connections Discovery | Windows | ✅ Detected |
| T1110.001 | Brute Force | Windows | ✅ Detected |
| T1055 | Process Injection | Windows | ✅ Detected |
| T1112 | Modify Registry | Windows | ✅ Detected |
| T1059.004 | Linux Shell Execution | Ubuntu | ✅ Detected |
| T1087 | Linux Account Discovery | Ubuntu | ✅ Detected |
| T1110.001 | SSH Brute Force | Ubuntu | ✅ Detected |

---

## Custom Detection Rules

- `rules/sigma/sigma_ssh_bruteforce.yml` — SSH brute force detection
- `rules/sigma/sigma_recon_passwd.yml` — /etc/passwd reconnaissance
- `rules/sigma/sigma_atomic_redteam.yml` — Atomic Red Team execution
- `rules/wazuh/local_rules.xml` — Custom Wazuh XML rules (IDs 100001–100003)

---

## Repository Structure
.
├── README.md
├── configs/
│   ├── wazuh/          # Wazuh manager + agent configs
│   └── sysmon/         # Sysmon XML configuration
├── rules/
│   ├── sigma/          # Custom Sigma detection rules
│   └── wazuh/          # Wazuh XML rules
├── playbooks/
│   └── detection-playbook.md
└── screenshots/        # Wazuh dashboard evidence
---

## Key Results

- 2 OS platforms monitored (Ubuntu + Windows Server)
- 11 MITRE ATT&CK techniques simulated and detected
- 400+ security events captured from Windows agent alone
- Sysmon providing deep process-level telemetry
- Custom Sigma and Wazuh rules for SSH brute force, recon, and ART execution

---

## Author

**Aykay05** — Cybersecurity student | Detection Engineering | SIEM/EDR  
Project completed as part of a hands-on cybersecurity training capstone.

All simulations performed in an isolated, self-owned AWS environment.
