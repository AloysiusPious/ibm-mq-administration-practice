# 📘 Day 14 - IBM MQ Pub/Sub Example — Topic-Based Messaging
This guide demonstrates topic-based Publish/Subscribe (Pub/Sub) in IBM MQ using a topic /SPORTS/TENNIS, 10 subscribers, and basic pub/sub commands.

### 🧠 What is Pub/Sub in MQ?
Publish/Subscribe is a decoupled messaging model:

|Role	|Description|
|------|-----------|
|Publisher |	Sends messages to a topic (not to a queue)|
|Subscriber	|Subscribes to the topic and receives a copy of the message|
|Topic |	Logical channel (e.g., /SPORTS/TENNIS) for routing messages|

All subscribers receive a copy of the published message if they match the topic.

### 🏗️ Architecture Overview
```bash
                    Publisher
                       │
                       ▼
           Topic: /SPORTS/TENNIS
         ┌─────┬─────┬─────┬─────┬─────┐
         ▼     ▼     ▼     ▼     ▼     ▼
        Q1    Q2    Q3    Q4    Q5    ...
        ▲     ▲     ▲     ▲     ▲
      SUB1  SUB2  SUB3  SUB4  SUB5 ...
```
Each subscription binds the topic string to a destination queue.

### ⚙️ Setup Commands
### 1️⃣ Define the Topic
``` bash
DEFINE TOPIC(SPORTS) TOPICSTR('/SPORTS/TENNIS')
```
This tells MQ that messages published to /SPORTS/TENNIS should follow the Pub/Sub engine.

### 2️⃣ Create Queues Q1 to Q10
``` bash
DEFINE QLOCAL(Q1)
DEFINE QLOCAL(Q2)
...
DEFINE QLOCAL(Q10)
```
Each queue will act as a receiver for one subscriber.

### 3️⃣ Define Subscriptions
``` bash
DEFINE SUB(SUB1) TOPICSTR('/SPORTS/TENNIS') DEST(Q1)
DEFINE SUB(SUB2) TOPICSTR('/SPORTS/TENNIS') DEST(Q2)
...
DEFINE SUB(SUB10) TOPICSTR('/SPORTS/TENNIS') DEST(Q10)
```
Each subscription links the topic to a specific queue.

### 🚀 Publish and Subscribe
### ✅ Publish a Message
``` bash
amqspub /SPORTS/TENNIS PUBSUBQMGR_ALOY
```
This command prompts for input.

Message will be routed to all subscribers of /SPORTS/TENNIS.

Important: The topic string must exactly match /SPORTS/TENNIS

✅ Receive Messages from Subscribers
``` bash
amqsget Q1 PUBSUBQMGR_ALOY
amqsget Q2 PUBSUBQMGR_ALOY
...
amqsget Q10 PUBSUBQMGR_ALOY
```
Each queue (Q1 to Q10) will contain a copy of the published message.

### 🔁 Behavior Demonstration
``` bash
Subscriber	Queue	Result After amqspub
SUB1	Q1	1 message received
SUB2	Q2	1 message received
...	...	...
SUB10	Q10	1 message received
```
✅ You successfully implemented 1-to-many fan-out delivery.

### 🧠 Notes
Multiple subscribers = multiple copies.

MQ automatically handles routing and duplication.

You can also create durable subscriptions or use wildcards (/SPORTS/#) for broader topics.

### 🧼 Cleanup
``` bash
DELETE SUB(SUB1)
...
DELETE SUB(SUB10)
DELETE QLOCAL(Q1)
...
DELETE QLOCAL(Q10)
DELETE TOPIC(SPORTS)
```
### 📚 References

[IBM MQ Pub/Sub Concepts](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=pubsub-publishsubscribe)

[amqspub Tool Usage](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=pubsub-publishsubscribe)
