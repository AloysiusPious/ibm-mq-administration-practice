# 📘 Day 6 - IBM MQ Practice

## 🗓️ Topic: Triggering a Program When a Message Arrives
### 🎯 Objective
Learn how to configure MQ Triggering so that when a message arrives on a queue, MQ automatically starts a program (trigger monitor launches it).

### This is useful for:
* Event-driven processing
* Batch jobs
* Reducing idle process overhead

### 📚 Key Concepts
| Term	| Description|
| -----| -----------|
| Triggering	| MQ feature to auto-start an application when a condition on a queue is met |
| Trigger Monitor | System process that listens to queues and launches target programs |
| INITQ | The initiation queue used by the trigger monitor |
| PROCESS object | Defines the program to be triggered |
| Trigger Type | FIRST, EVERY, or DEPTH — defines how frequently to trigger |

## 🛠️ Step-by-Step Practice
### ✅ 1. Define the Application Queue
```bash
runmqsc QM1
DEFINE QLOCAL('TRIGGER.TEST.Q') +
       TRIGGER +
       TRIGTYPE(FIRST) +
       TRIGDPTH(1) +
       TRIGMPRI(0) +
       TRIGDATA('TRIGGER.PROC') +
       INITQ('SYSTEM.DEFAULT.INITIATION.QUEUE')
END
```
✅ 2. Define the PROCESS Object
```bash
DEFINE PROCESS('TRIGGER.PROC') +
       DESCR('Trigger test process') +
       APPLTYPE(UNIX) +
       APPLICID('/opt/mqm/samp/bin/amqsget TRIGGER.TEST.Q QM1')
END
```
🔍 This will execute amqsget when the queue gets a message and is eligible to be triggered.

### ✅ 3. Start the Trigger Monitor
```bash
strmqtrm -m QM1 -q SYSTEM.DEFAULT.INITIATION.QUEUE
```
📌 This must run in the background or as a system service.

### ✅ 4. Send a Message to the Queue
```bash
echo "Hello Trigger" | /opt/mqm/samp/bin/amqsput TRIGGER.TEST.Q QM1
```
Expected behavior:

Trigger monitor notices a message on TRIGGER.TEST.Q

It checks that the queue was previously empty (TRIGTYPE = FIRST)

MQ uses TRIGDATA to match to the PROCESS

It starts amqsget and receives the message

### 🔎 5. Confirm Output
The message should be printed to the terminal where amqsget runs (might require redirecting logs if run as service).

🧹 Clean-Up
```bash
runmqsc QM1
DELETE QLOCAL('TRIGGER.TEST.Q')
DELETE PROCESS('TRIGGER.PROC')
END
```
Stop trigger monitor:

```bash
# Find PID
ps -ef | grep strmqtrm
```
# Kill it
```bash
kill <PID>
```
### 🔄 Optional Variations
TRIGTYPE(EVERY): Triggers on every message

TRIGTYPE(DEPTH): Triggers when queue depth reaches TRIGDPTH

Example:
```bash
ALTER QLOCAL('TRIGGER.TEST.Q') TRIGTYPE(DEPTH) TRIGDPTH(5)
```
### 📚 References
[IBM Docs — Triggering Concepts](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=triggering)
[IBM Docs — Trigger Monitor](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=monitors-trigger-monitor)

### 💡 Tips
Use custom scripts or services in the APPLICID of the PROCESS object.
Always test TRIGTYPE(FIRST) first to understand the behavior.
Ensure INITQ is correct and the monitor is started.

