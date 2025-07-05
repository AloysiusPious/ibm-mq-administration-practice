# 📘 Day-8: IBM MQ — Channels, SVRCONN, amqputc/amqgetc

This lesson focuses on how applications communicate with IBM MQ using **SVRCONN** channels and how to simulate client interactions using `amqputc` and `amqgetc`.

---

## 🔹 Objectives

- Understand what **SVRCONN** channels are
- Create a secure channel for client connections
- Simulate client connectivity using IBM MQ client tools
- Perform end-to-end messaging over TCP/IP

---

## 🔗 What is a SVRCONN Channel?

A **SVRCONN (Server-Connection)** channel is:
- Defined on the **MQ server**
- Used by client applications (or tools) to **connect remotely**
- Essential for all **TCP/IP based messaging**

---

## 🧠 Channel Types Overview

| Channel Type   | Purpose                                 | Used By             |
|----------------|------------------------------------------|---------------------|
| `SVRCONN`       | For client applications to connect       | MQ Clients          |
| `SDR` (Sender)  | For sending messages to remote QMGR      | Queue Manager       |
| `RCVR` (Receiver)| For receiving from remote QMGR          | Queue Manager       |
| `CLUSSDR`       | Cluster sender                           | Clustered QMGRs     |
| `CLUSRCVR`      | Cluster receiver                         | Clustered QMGRs     |

---

## 🛠️ Practical Setup

### ✅ 1. Create Local Queue on Server

```bash
DEFINE QLOCAL(TEST.QUEUE)
```
### ✅ 2. Define SVRCONN Channel
```bash
DEFINE CHANNEL(APP.SVRCONN) CHLTYPE(SVRCONN) TRPTYPE(TCP)
```
This allows client applications to connect using APP.SVRCONN.

### ✅ 3. Define & Start Listener
```bash
DEFINE LISTENER(LISTENER1) TRPTYPE(TCP) PORT(1414)
START LISTENER(LISTENER1)
```
Ensure port 1414 is open on your firewall.

#### 🌐 Client Machine Configuration
### ✅ 4. Set Connection Environment Variable
```bash
export MQSERVER="APP.SVRCONN/TCP/your-server-ip(1414)"
```
Replace your-server-ip with the MQ server's actual IP.

### ✅ 5. Send Message from Client
```bash
amqputc TEST.QUEUE QMGR
```
Type message content and press Enter

Press Ctrl+C to exit

### ✅ 6. Receive Message from Client
```bash
amqgetc TEST.QUEUE QMGR
```
#### ✅ Messages are read and removed from the queue.

## 🔐 Security & Access Control
### ✅ 7. Grant Access to Queue for Client User
```bash
setmqaut -m QMGR -t q -n TEST.QUEUE -p appuser +put +get +inq
```
### ✅ 8. Optional: Secure Channel with CHLAUTH (Map IP to MCAUSER)
```bash
SET CHLAUTH('APP.SVRCONN') TYPE(ADDRESSMAP) ADDRESS('192.168.1.100') MCAUSER('appuser')
```
|Field	|Description|
|-------|-----------|
|APP.SVRCONN	|Channel name|
|ADDRESS	|IP address of the client machine|
|MCAUSER	|User mapped to client connection|

## 🧪 Troubleshooting Tips
|Issue	|Cause	|Fix|
|-------|-------|---|
|AMQ9777 Channel was blocked	|CHLAUTH rule blocking IP	|Add ADDRESSMAP or disable CHLAUTH (test only)|
|2035 MQRC_NOT_AUTHORIZED	|Missing user authority	|Use setmqaut to allow access|
|Connection Refused	|Listener not started or port issue	|Start listener and open firewall port|
|amqputc hangs	|Wrong channel or server IP	|Double-check MQSERVER string|

## 🧠 Summary
|Component	|Description|
|-----------|-----------|
|SVRCONN channel	|Defines connection path for clients|
|MQSERVER var	T|ells client how to reach MQ|
|amqputc	|Sends test message (Client)|
|amqgetc	|Reads test message (Client)|
|CHLAUTH	|Restricts/controls client access by IP/user|
|setmqaut	|Grants authority to queue users|

### 📚 References
[IBM MQ: Channel Types](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=channels-channel-types)

[IBM MQ: Server-Connection Channels](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=types-server-connection-channel-svrconn)

[amqputc / amqgetc Utilities](https://www.ibm.com/docs/en/ibm-mq/9.3?topic=utilities-sample-programs)

