# 📘 Day-14: IBM MQ Cluster Namelist (CLUSNL / REPOSNL) — Explained with Practical Setup
## 🔹 1. What is a Namelist in IBM MQ?
A Namelist is a list of names grouped together under one name. It can include:

Queue names

Topic names

Queue manager clusters

In cluster context, Namelists are used for:

CLUSNL() → Control which clusters a queue is advertised in

REPOSNL() → Control which clusters a queue manager participates in

## 🔹 2. Real-Time Use Case for Namelist
In large environments, a queue manager may be part of multiple clusters (like ALPHA, BETA, GAMA). But not all queues need to be shared in all clusters.

So you:

Group the cluster names into a Namelist

Attach this list to queues (CLUSNL) or queue managers (REPOSNL)

This gives fine-grained control:

Who can see and put to a queue

What cluster visibility each queue has

## 🔹 3. Practical Setup Using Your Example
### 🏗️ Cluster Setup Summary
|QMGR	|Clusters Participated	|Port	|Type|
|------|----------------------|-----|-----|
|QMGR1	|ALPHA	|1111	|Full|
|QMGR2	|REPOSNL = IBMALOY	|2222	|Full|
Q|MGR3	|GAMA	|3333	|Full|

## 📝 Step-by-Step Configuration
### 🧩 Define Namelist
```bash
DEFINE NAMELIST(IBMALOY) NAMES(ALPHA, BETA, GAMA)
```
### 🛠️ On QMGR1 (ALPHA)
```bash
ALTER QMGR REPOS(ALPHA)

DEFINE CHANNEL(TO.QMGR2) CHLTYPE(CLUSSDR) CONNAME('localhost(2222)') CLUSTER(BETA)
DEFINE CHANNEL(TO.QMGR1) CHLTYPE(CLUSRCVR) CONNAME('localhost(1111)') CLUSTER(ALPHA)

DEFINE LISTENER(LSN.QMGR1) TRPTYPE(TCP) PORT(1111)
START LISTENER(LSN.QMGR1)
```
### 🛠️ On QMGR2 (Central Hub)
```bash
ALTER QMGR REPOSNL(IBMALOY)

DEFINE CHANNEL(TO.QMGR1) CHLTYPE(CLUSSDR) CONNAME('localhost(1111)') CLUSTER(ALPHA)
DEFINE CHANNEL(TO.QMGR2) CHLTYPE(CLUSRCVR) CONNAME('localhost(2222)') CLUSNL(IBMALOY)

DEFINE CHANNEL(TO.QMGR3) CHLTYPE(CLUSSDR) CONNAME('localhost(3333)') CLUSTER(GAMA)

DEFINE QLOCAL(Q1)
DEFINE QLOCAL(Q2)
DEFINE QLOCAL(Q3)

ALTER QL(Q1) CLUSNL(IBMALOY)
ALTER QL(Q2) CLUSNL(IBMALOY)
ALTER QL(Q3) CLUSNL(IBMALOY)

DEFINE LISTENER(LSN.QMGR2) TRPTYPE(TCP) PORT(2222)
START LISTENER(LSN.QMGR2)
```
🛠️ On QMGR3 (GAMA)
```bash
ALTER QMGR REPOS(GAMA)

DEFINE CHANNEL(TO.QMGR2) CHLTYPE(CLUSSDR) CONNAME('localhost(2222)') CLUSTER(BETA)
DEFINE CHANNEL(TO.QMGR3) CHLTYPE(CLUSRCVR) CONNAME('localhost(3333)') CLUSTER(GAMA)

DEFINE QLOCAL(QMGR3.LQ) CLUSTER(GAMA)

DEFINE LISTENER(LSN.QMGR3) TRPTYPE(TCP) PORT(3333)
START LISTENER(LSN.QMGR3)
```
### 🔄 4. How Does It Work?
✅ Queue Visibility Based on CLUSNL
CLUSNL(IBMALOY) is attached to Q1, Q2, Q3

IBMALOY contains ALPHA, BETA, GAMA

So, these queues are advertised to all 3 clusters

✅ Any queue manager in ALPHA, BETA, or GAMA can see and send messages to Q1–Q3

✅ Queue Manager Routing Based on REPOSNL
REPOSNL(IBMALOY) is set on QMGR2

QMGR2 becomes a gateway for all clusters in the namelist

Any app connected to QMGR2 can access queues in ALPHA, BETA, GAMA

### 📦 Message Flow Scenarios
|From	|To (Queue)	|Allowed?	|Reason|
|-----|-----------|---------|------|
|QMGR1	|Q1	|✅	|Both are in namelist via ALPHA|
|QMGR3	|Q3	|✅	|Both are in namelist via GAMA|
|QMGR1	|QMGR3.LQ	|❌	Q|MGR3.LQ is in GAMA only; QMGR1 not part of GAMA|
|QMGR2	|QMGR3.LQ	|✅	|QMGR2 participates in REPOSNL(IBMALOY) which includes GAMA|

### 🔐 Access Control
. MQ doesn’t restrict PUT access in cluster unless CLWLUSEREXIT or CHLAUTH is used

. You control visibility and routing using:

.. CLUSTER() — advertise to 1 cluster

.. CLUSNL() — advertise to multiple clusters

.. REPOSNL() — gateway behavior

### 🧠 Summary
Concept	You Used
Namelist	DEFINE NAMELIST(IBMALOY)
Queue CLUSNL	ALTER QL(Q1) CLUSNL(IBMALOY)
QMGR REPOSNL	ALTER QMGR REPOSNL(IBMALOY)
Multiple Clusters	ALPHA, BETA, GAMA

### 📚 References
[IBM MQ Cluster Namelist Doc[(https://www.ibm.com/docs/en/ibm-mq/9.3?topic=clusters-cluster-namelist)

[Using CLUSNL & REPOSNL](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=clusters-configuring-cluster)
