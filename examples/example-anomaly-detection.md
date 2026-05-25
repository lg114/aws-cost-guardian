# Example: Cost Anomaly Detection

## User Message

"Analyze my AWS costs for the last 30 days and identify any anomalies."

## AI Response

## Step 1: Cost Anomaly Report

| Metric | Current Month | Previous Month | Change | Status |
|--------|---------------|----------------|--------|--------|
| Total Cost | $1,200 | $500 | +140% | ⚠️ Anomaly |
| EC2 | $800 | $200 | +300% | ⚠️ Anomaly |
| S3 | $150 | $120 | +25% | ✅ Normal |
| RDS | $200 | $150 | +33% | ⚠️ Anomaly |
| Other | $50 | $30 | +67% | ✅ Normal |

### Anomalies Detected

1. **EC2 costs increased by 300%** — Root cause: 3 new m5.2xlarge instances launched on May 5th in us-east-1. These instances have been running at <5% CPU utilization since launch.

2. **RDS costs increased by 33%** — Root cause: A new db.r5.large instance was created on May 8th but shows 0 connections.

## Step 2: Service Analysis

### EC2 Analysis
| Instance ID | Type | CPU (14-day avg) | Status | Monthly Cost |
|-------------|------|------------------|--------|--------------|
| i-0abc123 | m5.2xlarge | 1.2% | Idle | $200 |
| i-0def456 | m5.2xlarge | 0.8% | Idle | $200 |
| i-0ghi789 | m5.2xlarge | 1.5% | Idle | $200 |

### RDS Analysis
| Instance ID | Class | CPU | Connections | Status | Monthly Cost |
|-------------|-------|-----|-------------|--------|--------------|
| prod-db-new | db.r5.large | 0% | 0 | Idle | $150 |

## Step 3: Optimization Plan

### Priority 1: Quick Wins (Low Risk, High Impact)
1. Stop idle EC2 instances — Est. savings: $600/month — Risk: Low
2. Stop idle RDS instance — Est. savings: $150/month — Risk: Low

**Total Estimated Savings: $750/month ($9,000/year)**

## Step 4: Remediation Scripts

### Option A: AWS CLI (Quick)
```bash
#!/bin/bash
# AWS Cost Guardian — Remediation Script
# Generated: 2024-05-25
# Total Estimated Savings: $750/month
# Risk Level: Low

# === Step 1: Stop idle EC2 instances ===
aws ec2 stop-instances --instance-ids i-0abc123 i-0def456 i-0ghi789
# These instances have <5% CPU for 20 days

# === Step 2: Stop idle RDS instance ===
aws rds stop-db-instance --db-instance-identifier prod-db-new
# This instance has 0 connections

# === Rollback ===
# aws ec2 start-instances --instance-ids i-0abc123 i-0def456 i-0ghi789
# aws rds start-db-instance --db-instance-identifier prod-db-new
```

### Option B: CloudFormation (Production)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — Stop Idle Resources'

Resources:
  # Stop idle EC2 instances
  EC2StopPolicy:
    Type: 'AWS::IAM::Policy'
    Properties:
      PolicyName: 'StopIdleEC2'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action: 'ec2:StopInstances'
            Resource:
              - 'arn:aws:ec2:*:*:instance/i-0abc123'
              - 'arn:aws:ec2:*:*:instance/i-0def456'
              - 'arn:aws:ec2:*:*:instance/i-0ghi789'

  # Stop idle RDS instance
  RDSStopPolicy:
    Type: 'AWS::IAM::Policy'
    Properties:
      PolicyName: 'StopIdleRDS'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action: 'rds:StopDBInstance'
            Resource: 'arn:aws:rds:*:*:db:prod-db-new'
```

### Option C: Terraform (Infrastructure)
```hcl
# AWS Cost Guardian — Stop Idle Resources
# Generated: 2024-05-25
# Total Estimated Savings: $750/month

# Stop idle EC2 instances
resource "aws_ec2_instance_state" "idle_instances" {
  for_each = toset(["i-0abc123", "i-0def456", "i-0ghi789"])
  instance_id = each.value
  state       = "stopped"
}

# Stop idle RDS instance
resource "aws_rds_cluster_instance" "idle_rds" {
  identifier         = "prod-db-new"
  cluster_identifier = aws_rds_cluster.main.id
  instance_class     = "db.r5.large"
  engine             = "aurora-postgresql"
}
```

## Step 5: Monitoring Setup

```bash
# Create billing alarm for $1,000 threshold
aws cloudwatch put-metric-alarm \
  --alarm-name "AWS-Cost-Alert-1000" \
  --alarm-description "Alert when AWS costs exceed $1,000" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 1000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:cost-alerts
```

## Step 6: Summary Report

### Analysis Period
- Start: 2024-04-25
- End: 2024-05-25
- Total Cost Analyzed: $1,200

### Findings
- 2 anomalies detected
- 4 idle resources identified
- $750/month potential savings

### Total Potential Savings
- Monthly: $750
- Annual: $9,000
- Percentage: 62.5%
