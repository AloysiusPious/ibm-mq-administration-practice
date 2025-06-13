# 📘 Day 2 - IBM MQ Practice

# 🗓️ Day 1: Setting Up Your First Queue Manager
🎯 Objective
Establish a foundational IBM MQ environment by creating a queue manager, defining a local queue, and verifying message operations.​

## 🛠️ Prerequisites
IBM MQ 9.4.0.10 installed on CentOS

User with mqm group privileges

Environment variables configured using setmqenv​

### ✅ Steps
## 1. Create a Queue Manager
The -q flag only tells MQ to create a quiescing-capable queue manager (which is the default behavior anyway).
```bash
crtmqm -q QM1
crtmqm -ld /mqdata/logs -md /mqdata/qmgrs QM1
```
Log files go to: /mqdata/qm1/log
You can also control the data directory (where queues/config are stored) using -md:

After Creation: Can You Move It?
After creating the queue manager:

❌ You cannot change the log directory via MQ commands.

✅ You can recreate the queue manager with the desired path.

Or move files manually (complex and risky — not recommended in prod without IBM support).

## 2. Start the Queue Manager
```bash
strmqm QM1
```
## 3. Define a Local Queue
``` bash
runmqsc QM1
DEFINE QLOCAL('TEST.QUEUE')
END
```
## 4. Display the Queue
```bash
runmqsc QM1
DISPLAY QLOCAL('TEST.QUEUE')
```
## 5. Send a Test Message
if /opt/mqm/samp/bin/amqsput path is not set in env variable then export it
```bash
echo "Hello MQ" | amqsput TEST.QUEUE QM1
```
After executing, type your message and press Enter. To end input, press Ctrl+D.​

## 6. Retrieve the Message
```bash
amqsget TEST.QUEUE QM1
```
This command reads the message from the queue.​

##7. Stop the Queue Manager
```bash
endmqm QM1
```
### 📘 Explanation
#Queue Manager (QM1): Central component managing queues and channels.

#Local Queue (TEST.QUEUE): Stores messages for applications to retrieve.

#amqsput and amqsget: Sample programs to put and get messages, useful for testing.​

## 📚 Additional Resources
[IBM MQ 9.4 Quick Start Guide](https://www.ibm.com/docs/en/ibm-mq/9.4.x?topic=mq-94-quick-start-guide)

[IBM MQ 9.4 Scenarios PDF](https://public.dhe.ibm.com/software/integration/wmq/docs/V9.4/PDFs/mq94.scenarios.pdf)


