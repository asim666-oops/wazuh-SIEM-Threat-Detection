# 🛡️ SOC Investigation Report  
## Brute Force Attack Detection using Wazuh SIEM  

---

## 📌 Incident Overview  

A suspicious authentication activity was detected on a Windows endpoint monitored by Wazuh SIEM. The alert was triggered due to multiple failed login attempts targeting the built-in Administrator account.  

The activity was generated as part of a controlled lab simulation to emulate a brute force attack scenario and validate the effectiveness of detection mechanisms.

---

## 🎯 Objective  

The objective of this investigation is to:  

- Detect failed authentication attempts  
- Analyze attack patterns using Wazuh logs  
- Identify indicators of brute force behavior  
- Correlate Windows Security Events with SIEM alerts  
- Map the activity to MITRE ATT&CK framework  

---

## ⚙️ Attack Simulation Performed  

The following commands were executed on the Windows victim machine:

### Step 1 — Enable Administrator Account
   
   net user Administrator P@ssword123 /active:yes

   
### Step 2 — Simulate Brute Force Attack

  for /L %i in (1,1,10) do net use \localhost\IPC$ /user:Administrator wrongpassword 2>nul for local 

  for /L %i in (1,1,20) do net use \\127.0.0.1\IPC$ /user:Administrator wrongpass
  for failed 

  
This loop generated multiple authentication failures in a short time frame, mimicking an automated brute force attack.

---

## 🚨 Alert Triggered in Wazuh  

- **Rule ID:** 60122  
- **Description:** Logon Failure - Unknown user or bad password  
- **Rule Level:** 5  
- **MITRE Mapping (Default):** T1531 (Account Access Removal)  

However, based on behavioral analysis, the activity aligns more accurately with:

- **Correct MITRE Technique:** T1110 – Brute Force  
- **Tactic:** Credential Access  

---

## 🧾 Log Analysis  

### 🔍 Event Details  

- **Event ID:** 4625 (Failed Logon)  
- **Agent Name:** WIN_VICTIM  
- **Agent IP:** 192.168.56.109  
- **Timestamp:** Mar 27, 2026  

---

### 🔐 Authentication Information  

- **Target Username:** Administrator  
- **Authentication Package:** NTLM  
- **Logon Process:** NtLmSsp  
- **Logon Type:** 3 (Network Logon)  

This confirms that the login attempts were made over the network using NTLM authentication.

---

### 🌐 Network Information  

- **Source IP Address:** ::1 (IPv6 loopback)  
- **Source Port:** 50409  
- **Workstation Name:** DESKTOP-0UJQ90E  

The source IP (::1) indicates that the attack originated from the same machine (local simulation).

---

### ❌ Failure Analysis  

- **Failure Reason:** Unknown user name or bad password  
- **Status Code:** 0xC000006D → Authentication failure  
- **Sub Status Code:** 0xC000006A → Incorrect password  

This confirms that the attack involved repeated incorrect password attempts rather than invalid usernames.

---

### 🔁 Attack Frequency  

- **Rule Fired Times:** 26  

Multiple alerts were triggered within a short duration, indicating an automated brute force attempt rather than manual login failure.

---

## 🧠 Behavioral Analysis  

The investigation reveals a clear pattern of repeated login failures targeting a privileged account (Administrator). The rapid succession of attempts, combined with identical failure reasons and consistent logon type, strongly indicates a brute force attack.

Key Indicators:

- Repeated failed logins  
- Same target account (Administrator)  
- Short time interval between attempts  
- Network-based authentication attempts (Logon Type 3)  
- NTLM authentication usage  

---

## 🔗 Event Correlation  

Further investigation steps (recommended in real SOC environment):

- Check for successful logins (Event ID 4624) after failures  
- Monitor account lockout events (Event ID 4740)  
- Identify lateral movement attempts  
- Analyze process creation logs using Sysmon  

In this scenario, no successful login was observed, confirming the attack was unsuccessful.

---

## 🧬 MITRE ATT&CK Mapping  

| Tactic              | Technique ID | Technique Name |
|--------------------|-------------|----------------|
| Credential Access  | T1110       | Brute Force    |

---

## 🛡️ Impact Assessment  

- No successful compromise detected  
- No privilege escalation observed  
- No persistence mechanisms identified  

The attack remained at the reconnaissance/credential access stage.

---

## ✅ Conclusion  

The Wazuh SIEM successfully detected multiple failed authentication attempts using Rule ID 60122. The analysis confirms that the activity was a brute force attack simulation targeting the Administrator account using incorrect credentials.

The detection occurred at an early stage, preventing any unauthorized access or system compromise.

---

## 🔒 Recommendations  

- Disable or rename default Administrator account  
- Implement account lockout policies  
- Enforce strong password policies  
- Enable multi-factor authentication (MFA)  
- Monitor failed login thresholds  
- Integrate Sysmon for deeper visibility  

---


## 🧪 Lab Environment  

- **SIEM:** Wazuh OVA  
- **Endpoint:** Windows Victim Machine  
- **Attack Method:** Local brute force simulation using CMD  
- **Log Source:** Windows Security Event Logs  

---
