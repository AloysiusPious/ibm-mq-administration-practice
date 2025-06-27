# 📘 Day 13 - IBM MQ Practice

## 📘 IBM MQ Cluster Load Balancing & HA Setup
This guide explains how to configure High Availability + Load Balancing using CLWLUSEQ(ANY) and DEFBind(NOTFIXED) for a shared cluster queue (QMGR01.LQ) across six queue managers.

### 🧠 What You’ll Learn
How to configure queue-level load balancing

Difference between DEFBind(FIXED) vs DEFBind(NOTFIXED)

Real message routing & HA simulation in cluster ALPHA

### 🏗️ Cluster Setup Overview
```bash
QMGR	Type	Port	Role in Cluster
QMGR01	FR	1419	Cluster Member & Queue Host
QMGR02	FR	1420	Cluster Member & Queue Host
QMGR03	PR	1421	Cluster Member & Queue Host
QMGR04	PR	1422	Cluster Member & Queue Host
QMGR05	PR	1423	Cluster Member & Queue Host
QMGR06	PR	1424	Cluster Member & Queue Host
```
### 🧪 Cluster Queue QMGR01.LQ is Shared Across All QMGRs
You defined the same cluster queue on all 6 QMGRs:

```bash
echo "define ql(QMGR01.LQ) cluster(ALPHA)" | runmqsc QMGR02
...
echo "define ql(QMGR01.LQ) cluster(ALPHA)" | runmqsc QMGR06
```
This allows IBM MQ to load balance messages across all six hosts.

### 🔧 Step-by-Step Load Balancing Setup
### 1️⃣ Set Queue Manager Cluster Behavior
```bash
echo "alter qmgr clwluseq(any)" | runmqsc QMGR01
...
echo "alter qmgr clwluseq(any)" | runmqsc QMGR06
```
🔍 CLWLUSEQ(ANY) — Instructs MQ to consider all queue instances in the cluster for message distribution.

### 2️⃣ Set Queue Binding Mode
```bash
echo "alter ql(QMGR01.LQ) defbind(notfixed)" | runmqsc QMGR01
...
echo "alter ql(QMGR01.LQ) defbind(notfixed)" | runmqsc QMGR06
```
🔍 DEFBIND(NOTFIXED) — Queue binding happens at every PUT, so MQ can choose a different queue manager for every message (instead of sticking to one for a connection).

### 🔁 What Happens Now?
If an app sends 6 messages to QMGR01.LQ using any QMGR (e.g., via QMGR02), the messages are automatically spread across all 6 instances:

Message #	Queue Manager Selected
1	QMGR01
2	QMGR03
3	QMGR05
4	QMGR02
5	QMGR04
6	QMGR06

### ✅ MQ decides routing dynamically using cluster metadata.

### 🔄 Test the Load Balancing
Send messages from one QMGR:
```bash
amqsput QMGR01.LQ QMGR02
```
# Enter messages manually or script them
Check where the messages landed:
```bash
echo "dis ql(QMGR01.LQ) curdepth" | runmqsc QMGR01
echo "dis ql(QMGR01.LQ) curdepth" | runmqsc QMGR02
...
echo "dis ql(QMGR01.LQ) curdepth" | runmqsc QMGR06
```
✅ You should see messages distributed across all QMs, not just one.

### 🚨 Common Misconfigs to Avoid
Issue	Explanation
Messages go only to one QM	Likely DEFBind(FIXED) or CLWLUSEQ(QMGR) is used
CLUSRCVR channel has wrong CONNAME	MQ cannot route messages correctly
No cluster queue defined on a QMGR	It won't receive messages even if it’s in the cluster

### 🔐 Application Connection Strategy
To use this load-balanced setup:

App connects to any one QMGR in the cluster (e.g., QMGR02)

Sends messages to QMGR01.LQ (available on all 6 nodes)

MQ handles routing automatically.

Connection Example:
```bash
makefile
Copy
Edit
QMGR: QMGR02
CHANNEL: APP.SVRCONN
CONNAME: QMGR02_IP(1420)
Queue: QMGR01.LQ
```
✅ App is unaware of where the message lands — MQ decides based on current availability.

### 📚 References

[IBM MQ Cluster Load Balancing Guide](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=clusters-workload-management)

[DEFBind & CLWLUSEQ Explained](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=clusters-workload-management)
