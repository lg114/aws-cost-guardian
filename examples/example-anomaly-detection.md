# Example: Anomaly Detection

## User Message

"My AWS bill jumped from $500/month to $1,200/month this month. Can you help me figure out what happened?"

## Guardian Response

## Cost Analysis Summary

| Metric | Current Month | Previous Month | Change |
|--------|---------------|----------------|--------|
| Total Cost | $1,200 | $500 | +140% |
| EC2 | $800 | $200 | +300% |
| S3 | $150 | $120 | +25% |
| RDS | $200 | $150 | +33% |
| Other | $50 | $30 | +67% |

### Findings

1. **EC2 costs increased by 300%** — Root cause: 3 new m5.2xlarge instances launched on May 5th in us-east-1. These instances have been running at <5% CPU utilization since launch.

2. **S3 costs increased by 25%** — Root cause: A new bucket `app-logs-prod` has accumulated 2TB of logs in Standard storage with no lifecycle policy.

3. **RDS costs increased by 33%** — Root cause: A new db.r5.large instance was created on May 8th but shows 0 connections.

### Recommendations

1. **Stop idle EC2 instances** — Est. savings: $600/month
   - 3x m5.2xlarge at ~$200/month each
   - CPU utilization <5% for 20 days

2. **Apply S3 lifecycle policy** — Est. savings: $40/month
   - Move logs to Standard-IA after 30 days
   - Expire logs after 90 days

3. **Stop or delete idle RDS instance** — Est. savings: $150/month
   - db.r5.large with 0 connections

**Total Potential Savings: $790/month (66% reduction)**

### Remediation Script

```bash
#!/bin/bash
# AWS Cost Guardian — Remediation Script
# Generated: 2024-05-25
# Issue: Unexpected cost spike from $500 to $1,200/month
# Estimated Savings: $790/month
# Risk Level: Low (stop) / High (terminate)

# === Step 1: Stop idle EC2 instances ===
# These instances have <5% CPU for 20 days
aws ec2 stop-instances --instance-ids i-0abc123 i-0def456 i-0ghi789

# === Step 2: Apply S3 lifecycle policy ===
aws s3api put-bucket-lifecycle-configuration \
  --bucket app-logs-prod \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "MoveToIA",
      "Status": "Enabled",
      "Filter": {"Prefix": ""},
      "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}],
      "Expiration": {"Days": 90}
    }]
  }'

# === Step 3: Stop idle RDS instance ===
aws rds stop-db-instance --db-instance-identifier prod-db-new

# === Rollback (if needed) ===
# Restart EC2: aws ec2 start-instances --instance-ids i-0abc123 i-0def456 i-0ghi789
# Restart RDS: aws rds start-db-instance --db-instance-identifier prod-db-new
```
