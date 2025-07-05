# 📘 Day-11: IBM MQ Queue Aliases & Remote Queues

This lesson explores **Queue Aliases** and **Remote Queues** — essential for routing and abstraction in distributed MQ systems.

---

## 🔹 Objectives

- Understand **Queue Alias** and **Remote Queue** concepts
- Learn how to route messages across queue managers using **XMITQ**
- Decouple producers from actual target queues

---

## 📦 Queue Types in This Lesson

| Type            | Purpose                                                              |
|------------------|----------------------------------------------------------------------|
| **Local Queue**   | Stores messages directly                                             |
| **Alias Queue**   | Points to a Local/Remote queue (logical indirection)                |
| **Remote Queue**  | Represents a queue on another queue manager; uses a transmission queue |

---

## 🧪 Practical Scenario

### 💡 Goal

Send a message to `REAL.QUEUE` on **QMGR2** by simply putting it on `ALIAS.QUEUE` on **QMGR1**.

---

## 🏗️ Architecture Overview

```text
[App] → ALIAS.QUEUE → REMOTE.QUEUE → XMITQ → CHANNEL → QMGR2 → REAL.QUEUE
