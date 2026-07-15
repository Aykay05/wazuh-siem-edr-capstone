# Detection Playbook
## Wazuh SIEM/EDR Capstone Project

---

## 1. SSH Brute Force — T1110.001

**Threat Profile:** Attacker repeatedly attempts SSH logins with different passwords to gain unauthorized access.

**Log Sources:** Linux auth logs, Wazuh rule ID 100001

**Detection Logic:**
- 5+ failed SSH attempts within 60 seconds from same source IP
- Wazuh alert level 10

**Investigation Steps:**
1. Identify source IP from `data.srcip` field
2. Check if IP is internal or external
3. Review timeline of attempts
4. Check if any attempt succeeded

**False Positives:**
- Legitimate users mistyping passwords
- Automated deployment tools

**Response:**
- Block source IP in security group
- Force password reset if compromise suspected
- Review all successful logins from that IP

---

## 2. Credential File Access — T1003.008

**Threat Profile:** Attacker reads /etc/shadow or /etc/passwd to extract password hashes for offline cracking.

**Log Sources:** Auditd, Wazuh rule ID 100002

**Detection Logic:**
- Any process accessing /etc/shadow
- Wazuh alert level 12

**Investigation Steps:**
1. Identify process that accessed the file (`data.audit.exe`)
2. Check if process is legitimate (e.g. passwd, shadow-utils)
3. Review user context running the process
4. Check for subsequent network connections (exfiltration)

**False Positives:**
- System administrators performing audits
- Backup tools accessing credential files

**Response:**
- Isolate endpoint if unauthorized access confirmed
- Review all processes run by that user
- Check for lateral movement attempts

---

## 3. PowerShell Abuse / Mimikatz — T1059.001

**Threat Profile:** Attacker uses PowerShell to execute malicious scripts including credential dumping tools like Mimikatz.

**Log Sources:** Windows Event Logs, Sysmon EventID 1, Wazuh

**Detection Logic:**
- PowerShell execution with suspicious parameters
- Mimikatz keywords in command line
- Sysmon process creation events

**Investigation Steps:**
1. Review full command line from `data.win.eventdata.commandLine`
2. Check parent process (`data.win.eventdata.parentImage`)
3. Review network connections made by PowerShell
4. Check for files dropped on disk

**False Positives:**
- Legitimate administrative PowerShell scripts
- Software deployment tools

**Response:**
- Isolate endpoint immediately
- Collect memory dump for forensics
- Review all processes spawned by PowerShell

---

## 4. LSASS Credential Dumping — T1003.001

**Threat Profile:** Attacker accesses LSASS process memory to extract plaintext credentials and password hashes.

**Log Sources:** Sysmon EventID 10, Windows Security Event 4656, Wazuh

**Detection Logic:**
- Process accessing lsass.exe memory
- Non-system process reading LSASS
- Sysmon process access events

**Investigation Steps:**
1. Identify process accessing LSASS (`data.win.eventdata.sourceImage`)
2. Check if process is legitimate security tool
3. Review timing and frequency of access
4. Check for credential use after dump

**False Positives:**
- Legitimate AV/EDR solutions
- Windows Defender

**Response:**
- Isolate endpoint immediately
- Reset all credentials on that system
- Review authentication logs for credential reuse

- ## 5. System Information Discovery — T1082

**Threat Profile:** Attacker enumerates system information to plan next steps in an attack.

**Log Sources:** Sysmon EventID 1, Windows Event Logs, Wazuh

**Detection Logic:**
- Execution of systeminfo.exe
- Registry queries for system information
- Sysmon process creation events

**Investigation Steps:**
1. Check what process ran systeminfo (`data.win.eventdata.image`)
2. Review parent process context
3. Check for subsequent reconnaissance activity
4. Correlate with other discovery techniques

**False Positives:**
- IT administrators gathering system info
- Software inventory tools

**Response:**
- Monitor for follow-up lateral movement
- Review all discovery activity in same timeframe
- Check for data exfiltration attempts

---

## 6. Registry Modification — T1112

**Threat Profile:** Attacker modifies registry keys to establish persistence or disable security controls.

**Log Sources:** Sysmon EventID 13, Windows Event Logs, Wazuh

**Detection Logic:**
- Modifications to sensitive registry keys
- reg.exe execution with suspicious parameters
- Sysmon registry events

**Investigation Steps:**
1. Identify registry key modified (`data.win.eventdata.targetObject`)
2. Check what value was set
3. Determine if change affects security controls
4. Review process that made the change

**False Positives:**
- Software installations modifying registry
- Group Policy updates

**Response:**
- Revert unauthorized registry changes
- Investigate process that made the change
- Check for persistence mechanisms

---

## 7. Process Injection — T1055

**Threat Profile:** Attacker injects malicious code into legitimate processes to evade detection and maintain access.

**Log Sources:** Sysmon EventID 8, Windows Event Logs, Wazuh

**Detection Logic:**
- Unusual process creating remote threads
- Shellcode execution patterns
- Sysmon CreateRemoteThread events

**Investigation Steps:**
1. Identify source and target process
2. Review memory regions involved
3. Check network activity from injected process
4. Look for dropped files or persistence

**False Positives:**
- Legitimate software using process injection (AV, debuggers)

**Response:**
- Isolate endpoint immediately
- Terminate injected process
- Full forensic investigation of memory

---

*Playbook developed as part of Wazuh SIEM/EDR Capstone Project by Aykay05*
*All simulations performed in isolated AWS environment*
