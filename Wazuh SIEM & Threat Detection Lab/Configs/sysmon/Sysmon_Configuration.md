# 🛡️ Sysmon Installation and Integration with Wazuh SIEM

---

## 1.  Introduction

System Monitor (Sysmon) is a Windows system service from Microsoft Sysinternals that provides detailed event logging for system activities such as:

- Process creation  
- Network connections  
- File creation  
- Process access  

When integrated with Wazuh SIEM, Sysmon enhances threat detection by supplying high-quality telemetry for security monitoring and analysis.

---

## 2.  Objective

The objective of this setup is to:

- Install Sysmon on a Windows endpoint  
- Integrate it with Wazuh SIEM  
- Enable real-time monitoring, detection, and analysis  
- Simulate a SOC (Security Operations Center) lab environment  

---

## 3.  Sysmon Installation Procedure

###  Step 1: Download Sysmon

- Download Sysmon from the official Microsoft Sysinternals Suite  
- Extract the downloaded package  

---

###  Step 2: Use a Custom Configuration

Instead of default configuration, a custom configuration is used to:

- Improve logging efficiency  
- Reduce unnecessary noise  
- Focus on security-relevant events  

A widely used configuration file is obtained from:

SwiftOnSecurity Sysmon Config Repository  

- File name: sysmonconfig-export.xml  
- Place it in the same directory as sysmon64.exe  

---

###  Step 3: Install Sysmon

Run Command Prompt as Administrator and execute:

    sysmon64.exe -accepteula -i sysmonconfig-export.xml

---

###  Step 4: Update Configuration (if already installed)

    sysmon64.exe -c sysmonconfig-export.xml

---

## 4.  Verification of Sysmon Installation

### 🔍 Check Service Status

- Open: services.msc  
- Locate: Sysmon (System Monitor)  
- Ensure status is: Running  

---

###  View Logs in Event Viewer

Navigate to:

    Applications and Services Logs
     → Microsoft
       → Windows
         → Sysmon
           → Operational

---

###  Important Sysmon Event IDs

| Event ID | Description        |
|----------|--------------------|
| 1        | Process Creation   |
| 3        | Network Connection |
| 10       | Process Access     |
| 11       | File Creation      |

These logs provide deep visibility into system-level activities.

---

## 5.  Integration of Sysmon with Wazuh

###  Locate Wazuh Agent Configuration File

    C:\Program Files (x86)\ossec-agent\ossec.conf

---

###  Add Sysmon Log Monitoring

Inside the <ossec_config> section, add:

    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

---

### 🔄 Restart Wazuh Agent

    net stop wazuh-agent
    net start wazuh-agent

---

## 6.  Log Collection and Analysis in Wazuh

After integration:

- Sysmon logs are forwarded to the Wazuh Server  
- Logs can be viewed in Wazuh Dashboard → Security Events  

---

###  Filtering Logs

Use filters such as:

- Keyword: sysmon  
- Event IDs (e.g., 1, 3, 10, 11)  
- Rule groups related to Sysmon  

---

###  Benefits

- Provides detailed endpoint visibility  
- Supports threat detection and investigation  
- Enables behavior-based monitoring  

---

## 7.  Testing and Validation

###  Perform Basic Actions

- Open applications  
- Run command-line tools  
- Execute scripts  

---

###  Expected Result

- Events should appear in:
  - Event Viewer (Sysmon logs)  
  - Wazuh Dashboard  

✔️ Successful visibility confirms proper integration.

---

## 8.  Conclusion

The integration of Sysmon with Wazuh SIEM significantly enhances SOC capabilities by:

- Providing detailed endpoint telemetry  
- Detecting suspicious activities such as:
  - Unauthorized process execution  
  - Lateral movement  
  - Malware behavior  

This setup is a critical component in modern security operations and SOC environments.

---

##  End of Setup