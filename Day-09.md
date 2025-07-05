# 📘 Day-9: IBM MQ — Message Browsing vs Destructive Gets

This lesson explains the difference between **browsing** and **destructive getting** messages from IBM MQ queues.

---

## 🔹 Objectives

- Understand how to **browse** messages without removing them
- Learn how to **get** and remove messages from queues
- Practice using `amqsbcg`, `amqsget`, and `MQGET` modes

---

## 🧠 Core Concepts

| Method           | Description                                 |
|------------------|---------------------------------------------|
| **Destructive Get** | Reads and **removes** the message from the queue |
| **Browse**          | Reads the message without removing it     |

---

## 🔍 Browsing Messages

Browsing is useful when:
- You want to inspect messages before processing
- You’re debugging applications
- You want to re-read or replay messages

---

## 🧪 Tools for Testing

### 1️⃣ `amqsbcg` (Browse + Details)

```bash
amqsbcg Q1 QMGR
```
- Lists all messages with metadata
- Does not remove messages
- Useful for inspection

### 3️⃣ MQGET Modes (in Application APIs)

MQGET is the API call used by applications to **retrieve messages** from queues.

Below are the commonly used **MQGMO (Get Message Options)** flags:

| MQGMO Option               | Description                                                             |
|---------------------------|-------------------------------------------------------------------------|
| `MQGMO_BROWSE_FIRST`      | Browse the **first** message in the queue                               |
| `MQGMO_BROWSE_NEXT`       | Browse the **next** message in the queue                                |
| `MQGMO_MSG_UNDER_CURSOR`  | Perform a **destructive get** of the last browsed message               |
| `MQGMO_NO_WAIT`           | Return immediately if there is no message (non-blocking)                |
| `MQGMO_WAIT`              | Wait for a message (blocking mode)                                     |
| `MQGMO_CONVERT`           | Convert message data to local code page (text format handling)          |

These flags are often combined to control how messages are read, browsed, or deleted by programs.

---

## 📦 Example Workflow

### ✅ Step 1: Put Messages

Send some test messages:

```bash
amqsput Q1 QMGR
```
### ✅ Step 2: Browse Messages
Use amqsbcg to browse messages (non-destructive):

```bash
amqsbcg Q1 QMGR
```
#### ✅ Messages will be printed to terminal.

#### 🔒 Queue depth remains unchanged.

### ✅ Step 3: Destructive Get
Use amqsget to consume and delete messages:

```bash
amqsget Q1 QMGR
```
### ✅ Messages are removed after reading.

#### 🔍 You can verify by checking queue depth.

### ✅ Step 4: Verify Queue Depth
```bash
echo "DIS QLOCAL(Q1) CURDEPTH" | runmqsc QMGR
```
Action	Expected CURDEPTH
After Put	> 0
After amqsbcg	Unchanged
After amqsget	0

### 🔐 Required Permissions
The user must have proper authorities to access the queue.
```bash
setmqaut -m QMGR -t q -n Q1 -p appuser +put +get +browse +inq
```
### 🧠 Summary Table
|Tool/API	|Action Type	|Removes Message	|Use Case|
|---------|-------------|-----------------|--------|
|amqsput	|Put message	|N/A	S|end test messages|
|amqsbcg	|Browse	|❌ No	|Debugging, auditing|
|amqsget	|Destructive Get	|✅ Yes	|Application-like message consumption|
|MQGET + BROWSE	|Programmatic	|❌ No	|Selective processing without deletion|
|MQGET + default	|Programmatic	|✅ Yes	|App retrieves and deletes messages|

### 📚 References
[IBM MQ Application Programming Guide](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=applications-programming)

[MQGMO Constants & Options](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=reference-mqgmo-get-message-options)

[MQScripting Tools - amqsput, amqsget, amqsbcg](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=samples-sample-programs)

