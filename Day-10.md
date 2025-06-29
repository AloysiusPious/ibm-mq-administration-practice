# 📘 Day 10 - IBM MQ Practice

## 🗓️ Topic: Monitoring Queue Depth & Queue Full Alerts
### 🎯 Objective
Learn how to:

Monitor queue depth

Set thresholds to get alerts when a queue gets too full or too empty

Understand event queues and how MQ notifies administrators of queue-related conditions

### 📚 Concepts
Term	Description
CURDEPTH	Current number of messages in a queue
MAXDEPTH	Maximum number of messages allowed in a queue
QDEPTHHI, QDEPTHLO	High and low watermarks for queue depth
QDEPTHMAXEV	Enables/disables queue depth events
SYSTEM.ADMIN.QMGR.EVENT	Queue where MQ writes queue manager event messages

### 🛠️ Step-by-Step Practice
### 🔧 1. Define a Local Queue with Depth Limits
```bash
runmqsc QM1
DEFINE QLOCAL('DEPTH.MONITOR.Q') MAXDEPTH(5000) +
       QDEPTHHI(80) QDEPTHLO(20) +
       QDEPTHMAXEV(ENABLED) +
       MONQ(HIGH)
END
```
### 📌 Explanation:

Triggers an event if queue depth exceeds 80% (high) or drops below 20% (low)

MONQ(HIGH) enables queue monitoring

### 🔧 2. Enable Event Messages at Queue Manager Level
```bash
runmqsc QM1
ALTER QMGR MONQ(HIGH)
ALTER QMGR QDEPTHHI(80) QDEPTHLO(20) QDEPTHMAXEV(ENABLED)
END
```
🔍 These settings allow MQ to generate queue depth event messages.

### 🔧 3. Fill the Queue to Trigger High Depth Event
```bash
for i in {1..4500}; do echo "Test $i" | /opt/mqm/samp/bin/amqsput DEPTH.MONITOR.Q QM1; done
```
This pushes more than 80% of 5000 (i.e., 4000) → triggers high depth alert

### 🔎 4. Check for Event in Event Queue
```bash
/opt/mqm/samp/bin/amqsget SYSTEM.ADMIN.QMGR.EVENT QM1
```
✅ You should see something like:

```bash
Queue Depth High Event
Queue: DEPTH.MONITOR.Q
Current Depth: 4500
Threshold: 4000
```
### 🔧 5. Test Low Depth Event
Clear some messages:
```bash
/opt/mqm/samp/bin/amqsget DEPTH.MONITOR.Q QM1
```
Keep consuming until depth goes below 20% (1000 messages)

Then re-check event queue.

### 🔍 6. Monitor Queue Manually (Optional)
```bash
DISPLAY QSTATUS(DEPTH.MONITOR.Q) CURDEPTH MAXDEPTH
```
### 🧼 Clean Up
```bash
runmqsc QM1
DELETE QLOCAL('DEPTH.MONITOR.Q')
END
```
### 💡 Tips
Use tools like MQ Explorer, custom scripts, or monitoring platforms to visualize depth.

For production, connect SYSTEM.ADMIN.QMGR.EVENT to a log collector or monitoring alert system.

### 📚 Reference
[IBM Docs — Queue Depth Events](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=events-queue-depth)

[IBM Docs — Monitoring Queues](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=queues-monitoring)

🧭 Coming Up (Day 11): MQ Triggering — automatically start a program when a message arrives.

Let me know if you want this whole series packaged into a GitHub repo or downloadable ZIP.



