
# 📘 IBM MQ Cluster Setup — Full Guide
## 📘 Day 12 - IBM MQ Practice

Content for Day 12 goes here...
This guide explains how to set up a basic MQ Cluster using:

2 Full Repositories: QMGR01, QMGR02

4 Partial Repositories: QMGR03 to QMGR06

Cluster name: ALPHA

How applications connect to the cluster

🧠 What is an MQ Cluster?
An MQ Cluster allows multiple queue managers to:

Share queues automatically across members

Route messages without manually defining transmission queues or remote queues

Provide load balancing and high availability

🏗Cluster Topology Overview
```bash
             CLUSTER: ALPHA
        ┌─────────────────────┐
        │                     │
        │     QMGR01 (FR)     │◄────┐
        │     Port: 1419      │     │
        └─────────────────────┘     │
                ▲   ▲              │
                │   │              │
        ┌───────┘   └───────┐      │
        ▼                  ▼      │
  QMGR03 (PR)        QMGR04 (PR)  │
  Port: 1421         Port: 1422   │
                                   │
       ┌─────────────────────┐     │
       │                     │     │
       │     QMGR02 (FR)     │◄────┘
       │     Port: 1420      │
       └─────────────────────┘
                ▲
                │
        ┌───────┼────────────┐
        ▼                   ▼
  QMGR05 (PR)         QMGR06 (PR)
  Port: 1423          Port: 1424
 ```
FR = Full Repository: Stores complete cluster data.

PR = Partial Repository: Connects to cluster using FR.

⚙️ Setup Summary
🔹 Queue Manager Definitions
QMGR	Type	Port	REPOS Value
QMGR01	FR	1419	REPOS(ALPHA)
QMGR02	FR	1420	REPOS(ALPHA)
QMGR03	PR	1421	-
QMGR04	PR	1422	-
QMGR05	PR	1423	-
QMGR06	PR	1424	-

🛰️ Cluster Channel Configurations
🎯 1. Full Repo: QMGR01 ↔ QMGR02
On QMGR01:
bash
Copy
Edit
DEFINE CHANNEL(TO.QMGR02) CHLTYPE(CLUSSDR) CONNAME('localhost(1420)') TRPTYPE(TCP) CLUSTER(ALPHA)
DEFINE CHANNEL(TO.QMGR01) CHLTYPE(CLUSRCVR) CONNAME('localhost(1419)') TRPTYPE(TCP) CLUSTER(ALPHA)
On QMGR02:
bash
Copy
Edit
DEFINE CHANNEL(TO.QMGR01) CHLTYPE(CLUSSDR) CONNAME('localhost(1419)') TRPTYPE(TCP) CLUSTER(ALPHA)
DEFINE CHANNEL(TO.QMGR02) CHLTYPE(CLUSRCVR) CONNAME('localhost(1420)') TRPTYPE(TCP) CLUSTER(ALPHA)
🎯 2. Partial Repo Queue Managers → QMGR01
(Each PR needs a CLUSSDR to one FR, and a CLUSRCVR to receive traffic)

Example for QMGR03:
bash
Copy
Edit
DEFINE CHANNEL(TO.QMGR01) CHLTYPE(CLUSSDR) CONNAME('localhost(1419)') TRPTYPE(TCP) CLUSTER(ALPHA)
DEFINE CHANNEL(TO.QMGR03) CHLTYPE(CLUSRCVR) CONNAME('localhost(1421)') TRPTYPE(TCP) CLUSTER(ALPHA)
Repeat similarly for QMGR04–QMGR06 with respective ports.

📢 Listeners
Start listeners for each QMGR:

bash
Copy
Edit
DEFINE LISTENER(LSN.QMGRxx) TRPTYPE(TCP) PORT(14xx)
START LISTENER(LSN.QMGRxx)
📦 Queue Shared in Cluster
Create a cluster queue only on QMGR01:

bash
Copy
Edit
DEFINE QLOCAL(QMGR01.LQ) CLUSTER(ALPHA)
Now this queue is visible to all cluster members, and any of them can put messages to QMGR01.LQ.

🔗 How Applications Connect to MQ Cluster
✅ App connection strategy:
App connects to any one QMGR (commonly a PR), and sends messages to the cluster queue.

Example:
App connects to QMGR03 on port 1421 and sends to QMGR01.LQ.

Connection string:

bash
Copy
Edit
CONNAME('QMGR03_IP(1421)') CHANNEL('TO.QMGR03') QMGR('QMGR03')
Since the queue QMGR01.LQ is in the cluster and hosted on QMGR01, MQ internally routes the message from QMGR03 → QMGR01 without defining:

Remote Queue

Transmission Queue

Queue Manager Alias

✅ Message Flow Example
bash
Copy
Edit
# On QMGR03
amqsput QMGR01.LQ QMGR03
➡️ MQ automatically finds that QMGR01.LQ is hosted on QMGR01 and delivers the message through cluster channels.

🧠 Load Balancing
If the same queue is defined as cluster queue on multiple QMs, MQ will load-balance puts across those.

In this example, only QMGR01.LQ exists on QMGR01 — so all traffic routes to it.

🔐 Security Note
For real applications:

Enable MCAUSER on CLUSRCVR channels.

Apply CHLAUTH rules to restrict who can connect.

Use TLS if connecting across hosts.

🧼 Cluster Cleanup (if needed)
To remove a queue manager from cluster:

bash
Copy
Edit
RESET CLUSTER(ALPHA) QMTYPE(QMGR) QMNAME(QMGR03)
📚 References
IBM MQ Clusters Overview

Cluster Queue Manager Connections

