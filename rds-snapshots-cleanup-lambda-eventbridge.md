This is continue with previous,.......

*To impliment automating the process (using Lambda + EventBridge) to sweep and delete expired manual snapshots on a daily schedule.*
# 🚀 What we are building

```text
Manual Snapshot (with ExpiryDate tag)
                ↓
EventBridge (runs daily)
                ↓
Lambda function
                ↓
Deletes expired snapshots automatically
```

---

# 🪜 STEP 1 — Tag a Manual Snapshot (Test First)

👉 We’ll test before automation.

### Go to:

```
AWS Console → RDS → Snapshots → Manual
```

1. Select any snapshot (or create one)
2. Click **Manage tags**
3. Add:

```text
Key: ExpiryDate       Value: 2026-04-09   (yesterday for testing)
Key: RetentionReason Value:  testing
```

✅ This snapshot should be deleted by Lambda later

---

# 🪜 STEP 2 — Create IAM Role for Lambda

### Go to:

```
IAM → Roles → Create role
```

1. Select:

```
Trusted entity → AWS service → Lambda
```

2. Click **Next → Create policy**

3. Paste this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBSnapshots",
        "rds:ListTagsForResource",
        "rds:DeleteDBSnapshot"
      ],
      "Resource": "*"
    }
  ]
}
```

4. Save policy → attach to role

5. Name role:

```text
rds-snapshot-cleanup-role
```

✅ Done

---

# 🪜 STEP 3 — Create Lambda Function

### Go to:

```
Lambda → Create function
```

Fill:

| Field   | Value                      |
| ------- | -------------------------- |
| Name    | `rds-snapshot-cleanup-` |
| Runtime | Python 3.12                |
| Role    | Use existing role          |

Click **Create**

---

# 🪜 STEP 4 — Add Lambda Code

Paste this (clean + production-ready):

```python
import boto3
from datetime import datetime, timezone

rds = boto3.client('rds')

def lambda_handler(event, context):
    snapshots = rds.describe_db_snapshots(SnapshotType='manual')['DBSnapshots']
    today = datetime.now(timezone.utc).date()

    for snap in snapshots:
        snapshot_id = snap['DBSnapshotIdentifier']
        db_id = snap['DBInstanceIdentifier']
        arn = snap['DBSnapshotArn']

        tags = rds.list_tags_for_resource(ResourceName=arn)['TagList']
        tag_dict = {t['Key']: t['Value'] for t in tags}

        expiry = tag_dict.get('ExpiryDate')

        if not expiry:
            print(f"Skipping {snapshot_id} (No ExpiryDate)")
            continue

        expiry_date = datetime.strptime(expiry, "%Y-%m-%d").date()

        if expiry_date < today:
            print(f"Deleting snapshot: {snapshot_id} (DB: {db_id})")
            rds.delete_db_snapshot(DBSnapshotIdentifier=snapshot_id)
```

👉 Click **Deploy**

---

# 🪜 STEP 5 — Test Lambda (VERY IMPORTANT)

### In Lambda:

Click **Test**

👉 Expected result in logs:

```text
Deleting snapshot: your-snapshot-name (DB: your-db)
```

Now go to:

```
RDS → Snapshots
```

✅ Snapshot should be deleted

---

# 🪜 STEP 6 — Create Daily Automation (EventBridge)

### Go to:

```
EventBridge → Rules → Create rule
```

Fill:

| Field     | Value                   |
| --------- | ----------------------- |
| Name      | `rds-cleanup-daily-` |
| Rule type | Schedule                |
| Schedule  | `rate(1 day)`           |

### Target:

* Select → Lambda
* Choose → `rds-snapshot-cleanup`

Click **Create**

---

# 🪜 STEP 7 — Configure  Backup Retention

### Go to:

```
RDS → Databases → Select DB → Modify
```

Set:

```text
Backup retention = 7 days
```

Click **Apply**

---

# ✅ FINAL TEST (IMPORTANT)

Create new snapshot:

```text
ExpiryDate = today or yesterday
```

Wait or manually run Lambda.

👉 Result:

✅ Snapshot auto deleted
✅ Logs available
✅ System working

---

# 🎯 What You Achieved

✅  snapshots auto-cleanup
✅ Cost control
✅ Policy compliance
✅ Fully automated system

---

# ⚠️ Small Safety Tip

If you want to be extra safe (recommended), modify code:

```python
if "prod" in db_id.lower():
    continue
```

👉 Prevents deleting production snapshots accidentally.

---

# ⭐ Final One-Line Understanding

👉 **You built an automatic system that deletes expired  RDS snapshots daily.**

---

# Prepared by:
*Shaik Moulali*
# DevOps Consultant
