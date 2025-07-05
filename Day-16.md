# 📘 Day-16: Persistent vs Non-Persistent Messages in IBM MQ

This lesson covers the difference between **persistent** and **non-persistent** messages in IBM MQ — how they are defined, how they behave during system failures, and how to configure them in queues and applications.

---

## 🎯 Objectives

- Understand the difference between persistent and non-persistent messages
- Know how MQ handles each type
- Learn how to configure persistence at queue or message level
- Test and verify message durability

---

## 1️⃣ What is Message Persistence?

| Type             | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **Persistent**   | Message survives MQ or system crash; stored to disk                         |
| **Non-Persistent** | Message is kept in memory; lost if QMGR crashes or restarts               |

---

## 2️⃣ Why Persistence Matters?

| Scenario                           | Persistent | Non-Persistent |
|------------------------------------|------------|----------------|
| Application sends banking txn logs | ✅ Required | ❌ Not safe     |
| Logging debug or monitoring data   | ❌ Overhead | ✅ Suitable     |
| Order placement system             | ✅ Required | ❌ Risky        |

---

## 3️⃣ How to Define Persistence?

### ✅ At Queue Level

When defining a queue, you can set a **default persistence** for messages placed on it:

```bash
DEFINE QLOCAL(Q.PERSIST) DEFPSIST(YES)
DEFINE QLOCAL(Q.NONPERSIST) DEFPSIST(NO)
```
### ✅ At Message Level (Application API)
In the code or test tool, the sender decides per message:

With amqsputc, the message is non-persistent by default.

In real applications, persistence is set via MQMD or MQPMO options.

#### 4️⃣ Verify Persistence Behavior
### ✅ Test Case
#### Define two queues:

```bash
DEFINE QLOCAL(Q.PERSIST) DEFPSIST(YES)
DEFINE QLOCAL(Q.NONPERSIST) DEFPSIST(NO)
```
#### Put messages:
```bash
amqsput Q.PERSIST QMGR
amqsput Q.NONPERSIST QMGR
```
#### Before consuming, simulate QMGR crash:

```bash
endmqm -i QMGR   # Immediate stop
strmqm QMGR
```
Get messages:

```bash
amqsget Q.PERSIST QMGR   # ✅ Messages still available
amqsget Q.NONPERSIST QMGR # ❌ Messages are lost
```
#### 5️⃣ MQMD Field: Persistence
| Constant                     | Value | Description                         |
| ---------------------------- | ----- | ----------------------------------- |
| `MQPER_PERSISTENT`           | 2     | Forces message to be persistent     |
| `MQPER_NOT_PERSISTENT`       | 1     | Forces message to be non-persistent |
| `MQPER_PERSISTENCE_AS_Q_DEF` | 0     | Uses queue default                  |

#### 6️⃣ Considerations
| Factor                 | Persistent | Non-Persistent    |
| ---------------------- | ---------- | ----------------- |
| Disk I/O               | Higher     | Lower (In-memory) |
| Delivery Guarantee     | Guaranteed | Best-effort       |
| Performance            | Slower     | Faster            |
| Used for Critical Data | ✅ Yes      | ❌ No              |
| Storage overhead       | ✅ More     | ✅ Less            |

#### 7️⃣ Monitoring & Audit
- Use dmpmqmsg or amqsbcg to view message headers and check persistence

- Use runmqsc to display queue and message stats

``` bash
DISPLAY QLOCAL(Q.PERSIST) CURDEPTH DEFSOPT DEFPSIST
```
## ✅ Summary
|Use Case	|Recommended |Persistence|
|---------|------------|-----------|
|Banking Transactions	|Persistent|
|Audit Logs	|Persistent|
|Health Check Pings	|Non-Persistent|
|Monitoring Alerts	|Non-Persistent|
|Order Placement Systems	|Persistent|

## 📚 References
[IBM MQ: Message Persistence](https://www.ibm.com/docs/en/ibm-mq/latest?topic=messages-persistence)

[MQMD Structure Documentation](https://www.ibm.com/docs/en/ibm-mq/latest?topic=structures-mqmd)
