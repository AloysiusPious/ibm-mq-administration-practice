# 📘 Day-18: IBM MQ — Backup & Restore of Entire Queue Manager Configuration

This lesson focuses on how to **backup** and **restore** the full configuration of an IBM MQ Queue Manager, including objects like queues, channels, listeners, and authorization records.

---

## 🔹 Objectives

- Learn to export MQ object definitions
- Understand how to backup MQ configuration and queue data
- Restore from backup in case of failure or migration
- Backup authorization records (setmqaut)
- Automate and schedule backups

---

## 📦 What Can Be Backed Up?

| Component              | Backup Tool/Command                     |
|------------------------|------------------------------------------|
| Object definitions     | `dmpmqcfg`                              |
| Queue data             | `saveqmgr`, `qload`, `MS03` supportpac |
| Authorization records  | `dmpmqcfg -a`, `setmqaut` commands      |
| Entire data directory  | `tar`, `rsync`, filesystem snapshot     |

---

## 🛠️ Backup Using `dmpmqcfg`

### ✅ Step 1: Backup All Object Definitions

```bash
dmpmqcfg -m QMGR -a > MQ_FULL_CONFIG.mqsc
-a includes authorization (setmqaut) commands
```
Output is in runmqsc format

You can later use this .mqsc script to recreate the entire configuration.

✅ Step 2: Backup to Separate Files (Optional)
```bash
dmpmqcfg -m QMGR -a -o object > objects.mqsc
dmpmqcfg -m QMGR -a -o authrec > auth.mqsc
```
#### 📥 Restore Using runmqsc
### ✅ Recreate Queue Manager (if needed)
bash
Copy
Edit
dltmqm QMGR
crtmqm QMGR
strmqm QMGR
### ✅ Run the Configuration Script
bash
Copy
Edit
runmqsc QMGR < MQ_FULL_CONFIG.mqsc
This restores all objects and permissions.

#### 📂 Backup Queue Data (Optional)
If queue contents (messages) are important, use:

### ✅ Option 1: qload (MS03 SupportPac)
```bash
qload -m QMGR -q Q1 -f Q1.backup
```
Backup multiple queues with scripting.

✅ Option 2: File System Snapshot (Cold Backup)
Stop the queue manager:

```bash
endmqm -w QMGR
```
Backup data directory:

```bash
tar -czvf /backup/QMGR_BACKUP.tar.gz /var/mqm/qmgrs/QMGR
```
Start the queue manager:

```bash
strmqm QMGR
```
Only valid if the queue manager is completely stopped before backup.

### 🧪 Verify Backup Files
### ✅ Check the object count
```bash
grep -c "DEFINE " MQ_FULL_CONFIG.mqsc
```
### 🧠 Best Practices
Schedule dmpmqcfg daily via cron:

```bash
0 2 * * * /opt/mqm/bin/dmpmqcfg -m QMGR -a > /backups/$(date +\%F)_QMGR.mqsc
```
- Store backups in version control (e.g., Git) or secure storage
- Backup before every major change
- Document dependencies like channels, listeners, ports

### 🔐 Permissions Needed
The user running dmpmqcfg should be part of the mqm group or have sufficient privileges.

### 📚 References
IBM MQ: dmpmqcfg Tool

SupportPac MS03 - qload

IBM MQ Backup and Recovery Guide

## ✅ Summary
|Task	|Tool	|Restorable?|
|----|-------|----------|
|MQ Object Config	|dmpmqcfg	✅ Yes|
|Queue Message Data	|qload	✅ Yes|
|Entire Filesystem	|tar/rsync	✅ Yes (cold)|
|Permissions/Auth	|dmpmqcfg -a	✅ Yes|
