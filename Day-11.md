# 📘 Day-11: IBM MQ Queue Aliases & Remote Queues

This lesson explores **Queue Aliases** and **Remote Queues** — essential for routing and abstraction in distributed MQ systems.

---

## 🔹 Objectives

- Understand **Queue Alias** and **Remote Queue** concepts
- Learn how to route messages across queue managers using **XMITQ**
- Decouple producers from actual target queues

---

## 📦 Queue Types in This Lesson

| Type            | Purpose                                                              |
|------------------|----------------------------------------------------------------------|
| **Local Queue**   | Stores messages directly                                             |
| **Alias Queue**   | Points to a Local/Remote queue (logical indirection)                |
| **Remote Queue**  | Represents a queue on another queue manager; uses a transmission queue |

---

## 🧪 Practical Scenario

### 💡 Goal

Send a message to `REAL.QUEUE` on **QMGR2** by simply putting it on `ALIAS.QUEUE` on **QMGR1**.

---

## 🏗️ Architecture Overview

```text
[App] → ALIAS.QUEUE → REMOTE.QUEUE → XMITQ → CHANNEL → QMGR2 → REAL.QUEUE
---

## 🛠️ Step-by-Step Setup

### 🧩 Environment Setup

| Component | Queue Manager | Purpose         |
|-----------|---------------|------------------|
| App Host  | QMGR1         | Sends message    |
| MQ Target | QMGR2         | Receives message |

---

### 🔹 Step 1: On QMGR2 (Target)

#### ✅ Define Local Queue

```bash
DEFINE QLOCAL(REAL.QUEUE)
✅ Start Listener (optional if already running)
bash
Copy
Edit
DEFINE LISTENER(LSN.TCP) TRPTYPE(TCP) PORT(1415)
START LISTENER(LSN.TCP)
🔹 Step 2: On QMGR1 (Source)
✅ Define Transmission Queue
This queue must match the name of the remote QMGR (QMGR2).

bash
Copy
Edit
DEFINE QLOCAL(QMGR2) USAGE(XMITQ)
✅ Define Remote Queue
This queue points to a queue on a remote QMGR and uses the transmission queue.

bash
Copy
Edit
DEFINE QREMOTE(REMOTE.QUEUE) RNAME(REAL.QUEUE) RQMNAME(QMGR2) XMITQ(QMGR2)
Field	Meaning
RNAME	Queue name on QMGR2 (REAL.QUEUE)
RQMNAME	Remote queue manager name (QMGR2)
XMITQ	Local transmission queue (QMGR2)

✅ Define Sender Channel
This channel sends messages from QMGR1 to QMGR2.

bash
Copy
Edit
DEFINE CHANNEL(TO.QMGR2) CHLTYPE(SDR) CONNAME('QMGR2_IP(1415)') XMITQ(QMGR2) TRPTYPE(TCP)
Replace QMGR2_IP with the actual hostname or IP of QMGR2.

Port should match QMGR2's listener.

✅ Define Alias Queue (Optional)
Create an alias that allows applications to send to ALIAS.QUEUE instead of knowing about REMOTE.QUEUE.

bash
Copy
Edit
DEFINE QALIAS(ALIAS.QUEUE) TARGQ(REMOTE.QUEUE)
This allows loose coupling between applications and the actual routing logic.

✅ Start the Sender Channel
bash
Copy
Edit
START CHANNEL(TO.QMGR2)
✅ Test the Flow
👉 Send a test message
bash
Copy
Edit
amqsput ALIAS.QUEUE QMGR1
You should see this message routed through:

pgsql
Copy
Edit
ALIAS.QUEUE ➜ REMOTE.QUEUE ➜ XMITQ ➜ TO.QMGR2 ➜ REAL.QUEUE
👉 Receive the message on QMGR2
bash
Copy
Edit
amqsget REAL.QUEUE QMGR2
You should see the exact message that was published from QMGR1.

yaml
Copy
Edit

---

Let me know if you'd like a **visual diagram**, **channel status script**, or **PDF cheat sheet** for these concepts.


