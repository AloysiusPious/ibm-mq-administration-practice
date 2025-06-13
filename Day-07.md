# 📘 Day 7 - IBM MQ Practice

## 🗓️ Topic: Dead Letter Queue (DLQ) & Message Recovery
### 🎯 Objective
Learn how MQ handles undeliverable messages using the Dead Letter Queue (DLQ), and how to use the Dead Letter Handler (runmqdlq) to recover or reroute these messages.

### 📚 Key Concepts
| Term | Description|
|------| -----------|
|DLQ	| A special queue for storing undelivered messages|
|runmqdlq	| IBM MQ utility for processing DLQ messages using rules|
|DLH	Dead | Letter Header, metadata attached to messages routed to the DLQ|
|REASON CODE	| Explains why a message was not delivered|

### 🛠️ Step-by-Step Practice
### ✅ 1. Identify the DLQ of a Queue Manager
``` bash
DISPLAY QMGR DEADQ
```
### ✅ Most default installations use:

SYSTEM.DEAD.LETTER.QUEUE

### ✅ 2. Send a Message to a Non-existent Queue
This simulates an undeliverable message:

``` bash
echo "Test for DLQ" | /opt/mqm/samp/bin/amqsput UNKNOWN.QUEUE QM1
```
❌ Since UNKNOWN.QUEUE does not exist, the message will go to the DLQ.

### 🔍 3. View Message in DLQ
``` bash
/opt/mqm/samp/bin/amqsget SYSTEM.DEAD.LETTER.QUEUE QM1
```
Output example:

``` bash
Dead Letter Header (DLH)
Reason Code: 2085 (MQRC_UNKNOWN_OBJECT_NAME)
Original Queue: UNKNOWN.QUEUE
Message: Test for DLQ
```
### ✅ 4. Create a Rule File for DLQ Handler
Create a file dlq.rules:

``` bash
vi /var/mqm/dlq.rules
```

Content:

``` bash
INPUTQM(QM1)
INPUTQ(SYSTEM.DEAD.LETTER.QUEUE)
RETRY(3)
REASON(2085) DISCARD
REASON(*) PUT(REAL.DEAD.STORAGE.Q)
```
This rule does:

* Discards unknown object name (2085) messages

* Moves other messages to REAL.DEAD.STORAGE.Q

### ✅ 5. Create the Target Queue for rerouting
``` bash
runmqsc QM1
DEFINE QLOCAL('REAL.DEAD.STORAGE.Q')
END
```
### ✅ 6. Run the DLQ Handler
``` bash
runmqdlq SYSTEM.DEAD.LETTER.QUEUE QM1 < /var/mqm/dlq.rules
```
🔄 This reads messages from DLQ and applies the rule file.

### 🔍 7. Check the Target Queue
``` bash
DISPLAY QLOCAL(REAL.DEAD.STORAGE.Q) CURDEPTH
```
If the message was moved, it will show depth > 0.

Retrieve it:

``` bash
/opt/mqm/samp/bin/amqsget REAL.DEAD.STORAGE.Q QM1
```
### 🧼 Clean-Up
``` bash
runmqsc QM1
DELETE QLOCAL('REAL.DEAD.STORAGE.Q')
END
```
### 💡 Tips
DLQ is crucial for production to prevent message loss.

Use runmqdlq with a CRON or as a service to automate recovery.

Always review the Reason Code in the DLH before reprocessing messages.

### 🧠 Common Reason Codes
|Code	|Meaning|
|-----|-------|
|2085	|MQRC_UNKNOWN_OBJECT_NAME|
|2035	|MQRC_NOT_AUTHORIZED|
|2053	|MQRC_Q_FULL|
|2058	|MQRC_Q_MGR_NAME_ERROR|

Full list: [IBM MQ Reason Codes](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=reference-mq-reason-codes)

### 📚 References
[IBM MQ — Dead Letter Queue Concepts](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=queues-dead-letter)

[runmqdlq Utility](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=utilities-runmqdlq-dead-letter-queue-handler)


