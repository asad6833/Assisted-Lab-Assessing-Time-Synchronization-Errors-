# ⏱️ Assisted Lab: Assessing Time Synchronization Errors  
**CompTIA CySA+ | Structureality Inc. | Time Sync, NTP, & Log Forensics**  
**Author: Asad Khan**

---

## 📘 Overview
Accurate timekeeping is essential in cybersecurity operations. Every event log, alert, and forensic artifact relies on proper timestamps to reconstruct an incident timeline.  
In this lab, you:

- Identify incorrect timestamps in Linux logs  
- Compare system clock vs hardware clock  
- Correct date, time, and time zone  
- Configure Linux NTP synchronization  
- Configure Windows Server (DC10) as a **stratum 1 NTP source**  
- Configure Windows Server (PC10) as a **stratum 2 NTP client**  
- Apply date offsets to historical logs for forensic accuracy  

---

## 🎯 Objectives
This lab supports CompTIA CySA+ objectives:

- **1.1** — System & network architecture in security operations  
- **4.2** — Incident response reporting & communication  

---

## 🖥 Environment
| System | Role |
|--------|------|
| **Kali Linux** | Log inspection, correction, & NTP client |
| **DC10 (Windows Server 2019)** | NTP stratum-1 server |
| **PC10 (Windows Server 2019)** | NTP stratum-2 client |
| **kern.log** | Kernel network logging (iptables) |

---

# 🔍 Part 1 — Identify Incorrect Log Timestamps
View the latest entries:
```bash
tail /var/log/kern.log
❗ Problem:
All log entries show the incorrect date 1971-04-01.

🕒 Part 2 — Compare Hardware Clock vs System Clock
Hardware clock:

bash
Copy code
hwclock
System clock:

bash
Copy code
date
Sync system clock to hardware clock:

bash
Copy code
hwclock -s
Verify:

bash
Copy code
hwclock && date
🌎 Part 3 — Set Correct Time Zone
View time zone options:

bash
Copy code
tzselect
Selected:

Copy code
America/Chicago
Apply:

bash
Copy code
timedatectl set-timezone America/Chicago
Check:

bash
Copy code
timedatectl
-0600 = time offset from UTC

🕰 Part 4 — Reset Date & Time (Kali)
Correct date:

bash
Copy code
date -s "3 Dec 2025"
Correct time:

bash
Copy code
date -s "22:16:00"
Confirm:

bash
Copy code
timedatectl
🧮 Part 5 — Apply Forensic Log Timestamp Correction
Incorrect date in logs:
1971-04-01

Correct date:
2025-12-03

Replace all timestamps:

bash
Copy code
sed -i 's/1971-04-01/2025-12-03/g' /var/log/kern.log
Verify:

bash
Copy code
grep 1971-04-01 /var/log/kern.log
tail /var/log/kern.log
🌐 Part 6 — Configure NTP on Kali
Enable NTP:

bash
Copy code
timedatectl set-ntp true
View sync status:

bash
Copy code
timedatectl timesync-status
Edit time source:

bash
Copy code
vim /etc/systemd/timesyncd.conf
Set:

ini
Copy code
NTP=203.0.113.227
Restart NTP:

bash
Copy code
timedatectl set-ntp false
timedatectl set-ntp true
Verify:

bash
Copy code
timedatectl show-timesync
hwclock && date && timedatectl
🕰 Part 7 — Configure DC10 as Stratum-1 NTP Server
Run PowerShell as Administrator:

powershell
Copy code
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\VMICTimeProvider -Name Enabled -Value 0;
w32tm /config /manualpeerlist:203.0.113.227 /syncfromflags:manual /reliable:yes /update;
shutdown /r /t 0;
After reboot:

Check source:

powershell
Copy code
w32tm /query /source
Expected:

Copy code
203.0.113.227
Check configuration:

powershell
Copy code
w32tm /dumpreg /subkey:parameters
Correct answer: w32time.dll

Status:

powershell
Copy code
w32tm /query /status
🖥 Part 8 — Configure PC10 as Stratum-2 NTP Client
Run PowerShell as Administrator:

powershell
Copy code
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\VMICTimeProvider -Name Enabled -Value 0
w32tm /config /manualpeerlist:DC10 /syncfromflags:manual /reliable:yes /update
Check time source (still old until resync):

powershell
Copy code
w32tm /query /source
Expected first value:

Copy code
203.0.113.227
Force sync:

powershell
Copy code
w32tm /resync
Check new source:

powershell
Copy code
w32tm /query /status
Expected:

makefile
Copy code
Source: DC10
Test auto-correction:

powershell
Copy code
Set-Date -Date 5/4/2025
w32tm /resync
🧠 Key Takeaways
✔ Proper time synchronization is critical for:
Log accuracy

Forensic reconstruction

SIEM correlation

Preventing false positives

Maintaining chain of custody

✔ Time Sources
Clock Type	Description
System Clock	OS-level, volatile
Hardware Clock (RTC)	Battery-backed, usually UTC
NTP Time	Network-synchronized, authoritative

✔ NTP Hierarchy Used
nginx
Copy code
Stratum 0 → External Reference (203.0.113.227)
Stratum 1 → DC10
Stratum 2 → PC10
Linux → NTP Client
✔ Forensic Best Practices
Never modify original log files

Always work on copies

Record all changes

Normalize to UTC when ingesting into SIEM


#ScreenShot
https://imgur.com/a/urc2bbb 
