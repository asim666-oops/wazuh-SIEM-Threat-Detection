# 🔍 File Integrity Monitoring (FIM) Investigation Steps using Wazuh

---

# 🔴 FILE CREATION — INVESTIGATION

## 🟢 Step 1 — Identify the Alert
Access the alert in Wazuh under:
- Integrity Monitoring / Security Events

Filter:
rule.groups: syscheck

📸 Screenshot 1:
Wazuh dashboard showing file creation alert

---

## 🟢 Step 2 — Review Alert Details
Open the alert and analyze:
- File Path
- Event Type → added
- Timestamp

🧠 Analysis:
- Check if the file location is normal or suspicious
- Example: C:\Users\Public → commonly abused location

📸 Screenshot 2:
Full alert details showing file path and event type

---

## 🟢 Step 3 — Evaluate File Characteristics
- File name → suspicious or normal?
- File type → .exe, .bat, .ps1 are high-risk

🧠 Analysis:
Executable files in public directories are suspicious

---

## 🟢 Step 4 — Check File History (Timeline)
Search:
data.syscheck.path: "C:\\Users\\Public\\filename.exe"

🔍 Observation:
- Confirm it is newly created
- Check if multiple events occurred

📸 Screenshot 3:
Timeline showing file creation event

---

## 🟢 Step 5 — Initial Conclusion
A new file was created in a potentially sensitive or publicly accessible location, which may indicate unauthorized file placement or malware drop activity.

---

# 🔴 FILE MODIFICATION — INVESTIGATION

## 🟢 Step 1 — Identify the Alert
Locate the file modification alert in Wazuh

📸 Screenshot 4:
Wazuh alert showing file modification

---

## 🟢 Step 2 — Review Alert Details
Open full log and analyze:
- File Path
- Event Type → modified
- Timestamp

📸 Screenshot 5:
Alert details view

---

## 🟢 Step 3 — Validate File Location & Type
- Check directory → normal or suspicious
- Check file type → executable/script

🧠 Analysis:
Modification of executable files in user-accessible directories is suspicious

---

## 🟢 Step 4 — Verify File Integrity (HASH CHECK)
From alert:
- Compare hash before and hash after

🔍 Observation:
If hash values differ → file content changed

📸 Screenshot 6:
Hash comparison (before vs after)

---

##  Step  — Final Conclusion
The file was modified with confirmed hash changes, indicating tampering. Based on its location and type, the activity is suspicious and may represent unauthorized modification or malicious behavior.

---

#  FINAL MEMORY FLOW
Alert → Details → Location → File Type → Hash → Timeline → Process → Conclusion