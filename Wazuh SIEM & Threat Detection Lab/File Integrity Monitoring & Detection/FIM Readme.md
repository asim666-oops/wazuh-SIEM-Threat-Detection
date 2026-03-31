#  File Integrity Monitoring (FIM) Hands-on Report using Wazuh (OSSEC)

##  Objective

The purpose of this lab is to understand how File Integrity Monitoring (FIM) works in a real SOC environment using Wazuh (OSSEC). The investigation focuses on detecting file creation and modification activities, analyzing alerts in the SIEM dashboard, and correlating those alerts with endpoint telemetry (Sysmon) to identify the process responsible for the activity.

---

##  Lab Setup

The lab environment consists of a Wazuh SIEM server and a Windows endpoint acting as the monitored agent (WIN_VICTIM). File Integrity Monitoring is configured on the endpoint to track changes within a specific directory. The Wazuh dashboard is used for SOC-level monitoring and investigation.

---

##  Step 1 — Configuring File Integrity Monitoring

The investigation begins by enabling File Integrity Monitoring on the Windows agent. The configuration file located at:

C:\Program Files (x86)\ossec-agent\ossec.conf  

is modified to include monitoring of the directory:

<syscheck>
  <frequency>30</frequency>

  <!-- Critical user activity -->
  <directories check_all="yes" realtime="yes">C:\Users</directories>

  <!-- System critical files -->
  <directories check_all="yes" realtime="yes">C:\Windows\System32</directories>

  <!-- Monitor temp (malware often drops here) -->
  <directories check_all="yes" realtime="yes">C:\Windows\Temp</directories>

  <!-- Ignore noisy files -->
  <ignore>C:\Windows\System32\*.log</ignore>

  <!-- Hash checking -->
  <check_sha1>yes</check_sha1>
  <check_sha256>yes</check_sha256>
</syscheck>

This is done by adding the `<syscheck>` configuration block. Once the configuration is updated, the Wazuh agent service is restarted to apply the changes.

📸 Screenshot 1:  
Configuration file showing the `<syscheck>` entry added for monitoring

---

##  Step 2 — Creating a Test File

To simulate suspicious activity, a test file is created inside the monitored directory using the command:

New-Item C:\Users\Public\ma1w@re.exe 

This action mimics the behavior of a potentially malicious file being dropped into a monitored location.

📸 Screenshot 2:  
Command prompt showing successful file creation

---

##  Step 3 — Detecting File Creation Alert
 
The SIEM successfully generates an alert indicating that a new file has been added. This confirms that FIM is actively monitoring the directory and detecting file creation events.

📸 Screenshot 3:  
Wazuh alert showing file creation (File added / Syscheck event)

---

##  Step 4 — Modifying the File

Next, the file is modified to simulate tampering or post-compromise activity. This is done using the command:

Add-Content C:\Users\Public\malware.exe "malicious code"  

This appends new content to the existing file, triggering a change in its integrity.

📸 Screenshot 4:  
Command prompt showing file modification

---

##  Step 5 — Detecting File Modification

The Wazuh dashboard is queried again using the filename:

ma1w@re.exe  

This time, the SIEM generates a file modification alert. The alert also includes a hash change, which is a critical indicator in File Integrity Monitoring. The hash difference confirms that the file content has been altered after its creation.

📸 Screenshot 5:  
Wazuh alert showing file modification and hash change

---

##  Step 6 — Alert Analysis (File Integrity Monitoring)

To analyze the file integrity alert, the investigation focused on the details provided by the FIM event in Wazuh.

The alert indicated that a file was modified in the following location:

Path: C:\Users\Public\ma1w@re.exe

During analysis, it was observed that:

The file is located in the Public directory, which is commonly abused by attackers to store malicious files.
The file name appears suspicious and resembles a malicious executable.
The event type was “modified”, indicating that the file content was altered after its creation.

To confirm whether the modification affected the file integrity, the hash values were examined:

MD5 (before change)
MD5 (after change)

It was found that:

👉 The MD5 hash values before and after the modification were different, confirming that the file content was successfully altered.

---

##  Analysis

The investigation confirms that Wazuh successfully detects both file creation and modification events in real time. The integrity of the file is validated using hash comparison, which changes upon modification. By correlating FIM alerts with Sysmon logs, it becomes possible to trace the exact process and user responsible for interacting with the file.

This reflects a real-world SOC workflow where an alert is generated, followed by investigation and correlation to validate whether the activity is benign or malicious.

---

##  Security Insight

File changes in critical or monitored directories can indicate serious security incidents such as malware execution, persistence mechanisms, or insider threats. FIM plays a crucial role in detecting hidden attacks, unauthorized modifications, and suspicious binaries that may otherwise go unnoticed.

---

##  Conclusion

This lab demonstrates the effectiveness of Wazuh File Integrity Monitoring in detecting and alerting on file changes. It also highlights the importance of correlating SIEM alerts with endpoint telemetry like Sysmon to gain full visibility into system activity. The exercise replicates a real SOC investigation process from detection to validation.

---

## Skills Gained

Through this lab, practical experience was gained in File Integrity Monitoring, Wazuh SIEM analysis, DQL-based log searching, SOC investigation workflows, and Sysmon-based correlation techniques.

---

##  Notes

FIM alerts should never be analyzed in isolation. Proper investigation always requires correlation with process logs and user activity. A single alert does not confirm an attack, but combined evidence can reveal the complete threat scenario.