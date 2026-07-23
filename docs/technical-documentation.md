# Technical Documentation
## Wazuh SIEM/EDR Capstone Project

---

## 1. Infrastructure Setup

### Wazuh Manager (EC2)
- **Instance type:** t3.small
- **OS:** Ubuntu 22.04 LTS
- **Region:** us-east-1d
- **Components:** Wazuh Manager 4.9.2, Wazuh Indexer, Wazuh Dashboard
- **Storage:** 50GB EBS

### Ubuntu Agent (EC2)
- **Instance type:** t3.micro
- **OS:** Ubuntu 22.04 LTS
- **Agent ID:** 001
- **Name:** ubuntu-agent-01

### Windows Agent (EC2)
- **Instance type:** c7i-flex.large
- **OS:** Microsoft Windows Server 2022 Datacenter
- **Agent ID:** 002
- **Name:** windows-agent-01

---

## 2. Network Configuration

### Security Groups

| Security Group | Port | Protocol | Source | Purpose |
|---|---|---|---|---|
| wazuh-manager-sg | 443 | TCP | My IP | Dashboard access |
| wazuh-manager-sg | 22 | TCP | My IP | SSH access |
| wazuh-manager-sg | 1514 | TCP | 172.31.0.0/16 | Agent communication |
| wazuh-manager-sg | 1515 | TCP | 172.31.0.0/16 | Agent enrollment |
| windows-agent-sg | 3389 | TCP | My IP | RDP access |

---

## 3. Agent Installation

### Ubuntu Agent
```bash
# Download and install Wazuh agent
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor > /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
apt-get update && apt-get install wazuh-agent

# Configure and start
WAZUH_MANAGER="172.31.46.149" systemctl start wazuh-agent
```

### Windows Agent
```powershell
# Download and install
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi -OutFile $env:TEMP\wazuh-agent.msi
msiexec.exe /i $env:TEMP\wazuh-agent.msi /q WAZUH_MANAGER="172.31.46.149" WAZUH_AGENT_NAME="windows-agent-01"

# Start service
NET START WazuhSvc
```

---

## 4. Sysmon Installation (Windows)

```powershell
# Download Sysmon
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "$env:TEMP\Sysmon.zip"
Expand-Archive -Path "$env:TEMP\Sysmon.zip" -DestinationPath "$env:TEMP\Sysmon" -Force

# Download SwiftOnSecurity config
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "$env:TEMP\sysmonconfig.xml"

# Install
& "$env:TEMP\Sysmon\Sysmon64.exe" -accepteula -i "$env:TEMP\sysmonconfig.xml"
```

### Wazuh Agent Config for Sysmon
```xml
<ossec_config>
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
</ossec_config>
```

---

## 5. Attack Simulations

### Atomic Red Team Installation (Windows)
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
Set-MpPreference -DisableRealtimeMonitoring $true
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force
```

### Simulations Run
```powershell
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force

Invoke-AtomicTest T1059.001 -TestNumbers 1  # Mimikatz
Invoke-AtomicTest T1087.002 -TestNumbers 1  # Account Discovery
Invoke-AtomicTest T1003.001 -TestNumbers 1  # LSASS Dump
Invoke-AtomicTest T1082 -TestNumbers 1      # System Info Discovery
Invoke-AtomicTest T1049 -TestNumbers 1      # Network Connections
Invoke-AtomicTest T1110.001 -TestNumbers 1  # Brute Force
Invoke-AtomicTest T1055 -TestNumbers 1      # Process Injection
Invoke-AtomicTest T1112 -TestNumbers 1      # Registry Modification
```

---

## 6. Custom Rules

### Wazuh XML Rules (IDs 100001-100003)
Located at: `/var/ossec/etc/rules/local_rules.xml`

| Rule ID | Level | Description | MITRE TTP |
|---|---|---|---|
| 100001 | 10 | SSH Brute Force Detection | T1110.001 |
| 100002 | 12 | /etc/shadow Access | T1003.008 |
| 100003 | 8 | Atomic Red Team Execution | T1059.004 |

---

## 7. Key Challenges & Solutions

| Challenge | Solution |
|---|---|
| Dynamic ISP IP blocking SSH | Updated security group inbound rules with current IP |
| Disk exhaustion on manager | Expanded EBS volume from 30GB to 50GB |
| JVM heap exhaustion | Upgraded from t3.micro to t3.small |
| Agent not connecting | Added ports 1514/1515 to wazuh-manager-sg |
| Windows agent enrollment failing | Fixed security group rules for VPC CIDR |

---

*Documentation by Aykay05 — Cybersecurity Capstone Project, July 2026*docs/technical-documentation.md
