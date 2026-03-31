# 🖥️ Windows 10 Virtual Machine Setup Using VirtualBox

---

## 1.  Introduction

A virtual machine (VM) is a software-based emulation of a physical computer that allows multiple operating systems to run on a single host system. In this project, a Windows 10 virtual machine is created using Oracle VM VirtualBox to serve as a victim system for security monitoring and attack simulation in a SOC lab environment.

---

## 2. Required Resources

The setup requires:

- VirtualBox installer (Oracle)
- VirtualBox Extension Pack
- Windows 10 ISO image (Microsoft)

These components are essential for creating and running the virtual machine.

---

## 3.  Installation of VirtualBox and Extension Pack

- Install VirtualBox using the setup wizard  
- Network drivers will be installed automatically  

After installation:

- Open VirtualBox  
- Go to Preferences → Extensions  
- Add the Extension Pack  

###  Benefits of Extension Pack:

- USB support  
- Improved graphics  
- Better host-guest integration  

---

## 4.  Creation of Windows 10 Virtual Machine

- Name: Windows 10 VM  
- Type: Microsoft Windows  
- Version: Windows 10 (64-bit)  

###  Memory Allocation

- Minimum: 4 GB  
- Recommended: 6–8 GB  

###  Storage Configuration

- Disk Type: VDI  
- Allocation: Dynamically allocated  
- Size: Minimum 50 GB  

---

## 5.  Advanced Virtual Machine Configuration

###  System Settings

- Disable Floppy  
- Boot Order: Optical → Hard Disk  
- CPU: Minimum 2 cores (Recommended 4)  
- Enable PAE/NX  

###  Display Settings

- Max video memory  
- Graphics Controller: VMSVGA  
- Enable hardware acceleration  

###  Storage

- Attach Windows 10 ISO file  

###  Network Configuration

- Adapter 1: NAT (Internet)  
- Adapter 2: Host-Only (Lab communication)  

---

## 6.  Installation of Windows 10

- Start VM  
- Select language, time, keyboard  
- Click Install  

###  Activation

- Select “I don’t have a product key”  

###  Edition

- Choose Windows 10 Pro  

###  Installation Type

- Custom Installation  
- Select unallocated space  

System installs and reboots automatically.

---

## 7.  Initial System Configuration

- Set region and keyboard  
- Create local offline account  
- Set username and password  

---

## 8.  Post-Installation Configuration

###  Install Guest Additions

- Devices → Insert Guest Additions CD  
- Run installer inside VM  
- Restart system  

###  Features Enabled

- Full-screen mode  
- Better resolution  
- Clipboard sharing  

###  Enable Features

- Clipboard: Bidirectional  
- Drag & Drop: Bidirectional  

---

## 9.  Network Configuration for SOC Lab

- Verify IP using command line  
- NAT → Internet  
- Host-Only → VM communication  

###  Enable Ping

- Allow ICMP in firewall  

---

## 10.  Remote Access Configuration

- Enable Remote Desktop (RDP)  
- Configure in System Properties  

---

## 11.  Snapshot Creation

- Take snapshot after setup  
- Used for restoring clean state  

---

# 🛡️ Wazuh Virtual Machine (OVA) Deployment

---

## 1.  Introduction

The Wazuh VM is a pre-configured SIEM environment distributed as an OVA file. It includes:

- Wazuh Manager  
- Indexer  
- Dashboard  
- Filebeat  

Runs on Amazon Linux 2023.

---

## 2.  Components Included

- Manager → Log analysis  
- Indexer → Storage & search  
- Dashboard → Visualization  
- Filebeat → Log forwarding  

---

## 3.  System Requirements

- 64-bit system  
- Virtualization enabled  

### Default VM Specs:

- CPU: 4 cores  
- RAM: 4 GB  
- Storage: 50 GB  

---

## 4.  Importing Wazuh OVA

- Open VirtualBox  
- Click Import Appliance  
- Select `.ova` file  

###  Modify Settings

- Graphics: VMSVGA  
- Enable UTC clock  

---

## 5.  Starting VM

- Start VM  

###  Login

- Username: wazuh-user  
- Password: wazuh  

###  Root Access

    sudo -i

---

## 6.  Accessing Wazuh Dashboard

Get IP:

    ip a

Access:

    https://<WAZUH_SERVER_IP>

###  Login

- Username: admin  
- Password: admin  

---

## 7.  Configuration Files

- Manager: /var/ossec/etc/ossec.conf  
- Indexer: /etc/wazuh-indexer/opensearch.yml  
- Filebeat: /etc/filebeat/filebeat.yml  
- Dashboard:
  - /etc/wazuh-dashboard/opensearch_dashboards.yml  
  - /usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml  

---

## 8.  Network Configuration

- Default: Bridged Adapter  
- Optional: Static IP  

---

## 9.  Time Synchronization

- Set hardware clock to UTC  
- Prevents log mismatch  

---

## 10.  Admin Access

- Use sudo instead of root login  

    sudo -i

---

# 💻 Wazuh Agent Deployment (Windows)

---

## 1.  Introduction

Wazuh agent collects endpoint data and sends it to the Wazuh manager for analysis.

---

## 2.  Purpose

- Monitor system activity  
- Detect threats  
- Centralized logging  

---

## 3.  Agent Package

- Use MSI installer (Windows)  
- Set server IP  
- Optional: Agent name & group  

---

## 4.  Installation

Run in PowerShell (Admin):

    Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.4-1.msi -OutFile $env:tmp\wazuh-agent
    msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='IP_ADDRESS'

---

## 5.  Start Agent Service

    NET START Wazuh

---

## 6. Agent Registration

- Connects automatically  
- Appears in dashboard  

---

## 7.  Configuration Flexibility

- Custom agent name  
- Group assignment  
- Advanced config options  

Used for:

- Intrusion detection  
- Threat analysis  
- Compliance monitoring  

---

##  End of Setup