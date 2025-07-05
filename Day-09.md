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
