# 🗓️ Day 14 - Topic: Message Expiry (Time-to-Live / TTL)
### 🎯 Objective
Learn how to configure and observe message expiry in IBM MQ — a feature that defines how long a message can remain in a queue before it is discarded.

### 📚 Key Concepts
|Term	|Description|
|-----|-----------|
|Expiry	|Message property indicating how long a message is valid|
|TTL (Time-to-Live)	|Time (in tenths of a second) that a message remains active before expiring|
|MQMD Expiry	|Field in the message descriptor that defines the message lifespan|
Expired Message	|Removed by MQ automatically — not moved to DLQ|

Note: Message expiry is based on time spent in the queue not during transit.

### 🛠️ Step-by-Step Practice
✅ 1. Create a Test Queue
``` bash
runmqsc QM1
DEFINE QLOCAL('EXPIRY.TEST.Q')
END
```
✅ 2. Send an Expiring Message (10 seconds expiry)
Use the amqsputc sample tool which allows you to modify message headers.

``` bash
/opt/mqm/samp/bin/amqsputc EXPIRY.TEST.Q QM1
```
When prompted, enter:

``` bash
Hello with expiry
```
📌 By default, amqsputc sends non-persistent messages with expiry = 10,000 (1,000 seconds).

To send a custom expiry:

Edit and compile your own version of amqsputc (advanced), or use an app/API that sets the MQMD.Expiry to a low value like 100 (10 seconds).

✅ 3. Wait for Message to Expire
After 10 seconds (or the expiry value you set), try to read from the queue:

``` bash
/opt/mqm/samp/bin/amqsget EXPIRY.TEST.Q QM1
```
👉 You will get:

``` bash
no more messages
```
✅ The message expired and was removed silently.

🧪 Advanced: Confirm Expiry Behavior
Try this test:

Put a message with short expiry (e.g., 5 seconds)

Quickly check queue depth:

``` bash
DISPLAY QLOCAL(EXPIRY.TEST.Q) CURDEPTH
```
Wait 10 seconds

Check queue depth again — should be 0

🔍 Note on DLQ and Expiry
Expired messages do NOT go to the Dead Letter Queue (DLQ). They are discarded silently by MQ.

### 🧼 Clean-Up
``` bash
runmqsc QM1
DELETE QLOCAL('EXPIRY.TEST.Q')
END
```
### 💡 Tips
Useful for TTL logic in temporary, retry, or event queues.

Can be used in combination with triggering for time-sensitive processes.

Always monitor expiry counts via queue statistics in production.

### 📚 References

[IBM MQ Message Expiry - Official Docs](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=mqmd-expiry-message)

[MQMD Fields Explained](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=descriptors-message-descriptor-mqmd)
