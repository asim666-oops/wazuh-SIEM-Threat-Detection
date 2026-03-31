# File Integrity Monitoring Attack Simulation

## Objective
To simulate unauthorized file activity and test Wazuh FIM detection.

## Actions Performed
- Created a file in monitored directory
- Modified file content
- Deleted the file

New-Item C:\Users\Public\ma1w@re.exe 
Add-Content C:\Users\Public\malware.exe "Script for reverse shell" 
rm ma1w@re.exe 

## Expected Detection
- Wazuh should trigger alerts for:
  - File creation
  - File modification
  - File deletion