**Sanpshots Cleanup**
1. Dev snapshots: 7-day automated retention; manual snapshots deleted within 30 days unless there is a specific reason to retain.
2. Production snapshots: 35-day automated retention; manual snapshots retained for up to 1 year where required for compliance, then deleted.
3. All manual snapshots must be tagged with an ExpiryDate and RetentionReason at the time of creation.
4. An automated process (Lambda + EventBridge) should  be implemented to sweep and delete expired manual snapshots on a daily schedule.

---

# 🧾 What This Policy Means (Simple English)

Your company defined **rules for database backups (RDS snapshots)**.

There are **two environments**:

* 🧪 **Dev (Development)**
* 🏭 **Production**

Each has different retention rules.

---

# ✅ 1️⃣ DEV Snapshots Policy

### Automated snapshots

> **7-day automated retention**

Meaning:

* AWS automatically takes backups.
* AWS automatically keeps them **7 days only**.
* After 7 days → AWS deletes them automatically.

✅ Nothing extra for you to build.

You just set:

```id="ebpjup"
Backup retention = 7 days
```

inside RDS settings.

---

### Manual snapshots (Dev)

> Deleted within 30 days unless required.

Meaning:

* If someone creates a manual backup,
* It must be deleted within **30 days**.

Unless there is a valid reason.

So every manual snapshot needs:

| Tag             | Example    |
| --------------- | ---------- |
| ExpiryDate      | 2026-05-01 |
| RetentionReason | Testing    |

---

# ✅ 2️⃣ PRODUCTION Snapshots Policy

### Automated snapshots

> **35-day automated retention**

Meaning:

AWS auto backups stay for **35 days**.

Set in RDS:

```id="x8rlfd"
Backup retention = 35 days
```

---

### Manual snapshots (Production)

> Keep up to **1 year** for compliance.

Meaning:

Some backups must stay longer for audits/legal reasons.

Example:

* Financial audit
* Release backup
* Migration checkpoint

But:

👉 After 1 year → MUST be deleted.

---

# ✅ 3️⃣ Mandatory Tags (VERY IMPORTANT)

Every manual snapshot MUST have:

| Tag             | Why            |
| --------------- | -------------- |
| ExpiryDate      | When to delete |
| RetentionReason | Why kept       |

Example:

```id="e90cbe"
ExpiryDate = 2027-04-01
RetentionReason = Compliance backup
```

Without tags = ❌ policy violation.

---

# ✅ 4️⃣ Automation Requirement (Main Work)

This is the part YOU must implement.

They want automatic cleanup.

---

## Architecture (Simple)

```id="6tq4z1"
EventBridge (daily timer)
          ↓
Lambda function
          ↓
Check snapshots
          ↓
Delete expired ones
```

---

## What happens daily

Every day AWS will:

1. Look at manual snapshots
2. Read `ExpiryDate`
3. If today > expiry date
4. Delete snapshot automatically

No manual work.

---

# 🎯 What YOU Must Configure (Actual Tasks)

## Task 1 — Configure Automated Backup Retention

### DEV DB

```
RDS → Database → Modify
Backup retention = 7 days
```

### PROD DB

```
Backup retention = 35 days
```

---

## Task 2 — Ensure Manual Snapshots Have Tags

When creating snapshot:

```
Create snapshot → Add tags
```

Add:

```
ExpiryDate
RetentionReason
```

---

## Task 3 — Create Auto Cleanup System

You must create:

| Service     | Purpose                   |
| ----------- | ------------------------- |
| Lambda      | Deletes expired snapshots |
| EventBridge | Runs daily                |

---

# 🧠 Real-Life Example

### Dev snapshot created today:

```
Date: Apr 9 2026
ExpiryDate: May 9 2026
```

After May 9:

✅ Lambda deletes automatically.

---

### Production compliance snapshot:

```
ExpiryDate: Apr 9 2027
Reason: Audit backup
```

After 1 year:

✅ Automatically removed.

---

# ✅ Final Policy Summary (One Screen)

| Environment | Auto Backup | Manual Backup      |
| ----------- | ----------- | ------------------ |
| DEV         | 7 days      | Delete ≤ 30 days   |
| PROD        | 35 days     | Keep ≤ 1 year      |
| ALL         | —           | Must have tags     |
| ALL         | —           | Auto cleanup daily |

---

# ⭐ One-Line Understanding

👉 **You are building an automatic expiry system for RDS manual backups based on company retention rules.**

---

# Prepared by:
*Shaik Moulali*
# DevOps COnsultant
