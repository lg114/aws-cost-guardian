# Example: Multi-Service Cost Optimization

## User Message

"Analyze my AWS environment and generate a comprehensive cost optimization report."

## AI Response

## Step 1: Cost Anomaly Report

| Service | Current Cost | Previous Month | Change | Status |
|---------|--------------|----------------|--------|--------|
| EC2 | $2,400 | $2,400 | 0% | ✅ Normal |
| S3 | $300 | $250 | +20% | ⚠️ Drift |
| RDS | $600 | $600 | 0% | ✅ Normal |
| Lambda | $50 | $50 | 0% | ✅ Normal |
| **Total** | **$3,350** | **$3,300** | **+1.5%** | ✅ Normal |

## Step 2: Service Analysis

### EC2 Analysis
| Instance | Type | CPU | Status | Recommendation | Savings |
|----------|------|-----|--------|----------------|---------|
| web-1 | m5.xlarge | 25% | Over-provisioned | Right-size to m5.large | $360/yr |
| web-2 | m5.xlarge | 25% | Over-provisioned | Right-size to m5.large | $360/yr |
| web-3 | m5.xlarge | 25% | Over-provisioned | Right-size to m5.large | $360/yr |

### S3 Analysis
| Bucket | Size | Class | Issue | Recommendation | Savings |
|--------|------|-------|-------|----------------|---------|
| app-logs | 5TB | Standard | Wrong class | Move to Standard-IA | $120/yr |

### RDS Analysis
| Instance | Class | CPU | Status | Recommendation | Savings |
|----------|-------|-----|--------|----------------|---------|
| prod-db | db.r5.xlarge | 15% | Over-provisioned | Right-size to db.r5.large | $180/yr |

## Step 3: Optimization Plan

### Priority 1: Quick Wins
1. Right-size EC2 instances — Est. savings: $720/year — Risk: Medium
2. Apply S3 lifecycle policy — Est. savings: $120/year — Risk: Low

### Priority 2: Reserved Instances
1. Purchase 1-year RI for EC2 — Est. savings: $720/year — Risk: Low
2. Purchase 1-year RI for RDS — Est. savings: $180/year — Risk: Low

**Total Estimated Savings: $1,740/year**

## Step 4: Remediation Scripts

### Option A: AWS CLI
```bash
#!/bin/bash
# AWS Cost Guardian — Multi-Service Optimization
# Total Estimated Savings: $1,740/year

# === EC2 Right-sizing ===
for i in web-1 web-2 web-3; do
  aws ec2 stop-instances --instance-ids $i
  aws ec2 modify-instance-attribute --instance-id $i --instance-type m5.large
  aws ec2 start-instances --instance-ids $i
done

# === S3 Lifecycle Policy ===
aws s3api put-bucket-lifecycle-configuration \
  --bucket app-logs \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "MoveToIA",
      "Status": "Enabled",
      "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}]
    }]
  }'
```

### Option B: CloudFormation
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — Multi-Service Optimization'

Resources:
  S3Lifecycle:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: app-logs
      LifecycleConfiguration:
        Rules:
          - Id: MoveToIA
            Status: Enabled
            Transitions:
              - TransitionInDays: 30
                StorageClass: STANDARD_IA
```

### Option C: Terraform
```hcl
# S3 Lifecycle Policy
resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = "app-logs"
  rule {
    id     = "move-to-ia"
    status = "Enabled"
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
  }
}
```

## Step 6: Summary Report

### Total Potential Savings
- Monthly: $145
- Annual: $1,740
- Percentage: 5.2%
