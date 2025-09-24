# Disaster Recovery Setup – Automated RDS Snapshots & Cross-Region Backup  
![AWS](https://img.shields.io/badge/AWS-RDS-orange)  
![Lambda](https://img.shields.io/badge/AWS-Lambda-blue)  
![EventBridge](https://img.shields.io/badge/AWS-EventBridge-purple)  

## 📌 Overview  
This project implements a **Disaster Recovery (DR) solution** using Amazon RDS, AWS Lambda, and EventBridge.  
It ensures automated **daily database backups**, **snapshot retention**, and **cross-region replication** for compliance and resiliency.  

Built entirely on **AWS Free Tier**, this solution is **cost-effective and production-ready**.  

---

## 🏗️ Architecture  
![Architecture Diagram](architecture-diagram.png)  

**Components:**  
- **Amazon RDS (Free Tier DB instance)** – primary database.  
- **AWS Lambda (Python + boto3)** – automates snapshot creation & cleanup.  
- **Amazon EventBridge** – schedules daily snapshot automation.  
- **Amazon CloudWatch** – monitors & logs execution.  
- **Amazon S3 (optional)** – exports snapshots for long-term archival.  

---

## ⚙️ Implementation Steps  
1. **Setup RDS Instance** – Launch a free-tier RDS instance (MySQL/Postgres).  
2. **Create Manual Snapshots** – Test backup & restore process.  
3. **Automate with Lambda** – Python function creates daily snapshots & retains last 3.  
4. **Schedule with EventBridge** – Cron-based schedule (e.g., midnight UTC).  
5. **Cross-Region Backup** – Copies snapshots to a secondary AWS region.  
6. **Monitor & Validate** – CloudWatch logs & alarms for execution tracking.  

---

## 🐍 Lambda Function Code (Python)  

```python
import boto3, datetime

rds = boto3.client('rds')

def lambda_handler(event, context):
    db_instance = "mydb"  # replace with your DB identifier
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d-%H-%M")
    snapshot_id = f"{db_instance}-snapshot-{timestamp}"

    # Create snapshot
    response = rds.create_db_snapshot(
        DBSnapshotIdentifier=snapshot_id,
        DBInstanceIdentifier=db_instance
    )
    print(f"Created snapshot: {snapshot_id}")

    # Retain only last 3 snapshots
    snapshots = rds.describe_db_snapshots(
        DBInstanceIdentifier=db_instance,
        SnapshotType='manual'
    )['DBSnapshots']

    if len(snapshots) > 3:
        snapshots_sorted = sorted(snapshots, key=lambda x: x['SnapshotCreateTime'])
        old_snap = snapshots_sorted[0]['DBSnapshotIdentifier']
        rds.delete_db_snapshot(DBSnapshotIdentifier=old_snap)
        print(f"Deleted old snapshot: {old_snap}")

    return response
