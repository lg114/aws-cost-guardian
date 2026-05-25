# AWS Cost Guardian — Cost Optimization Task

## Task

Analyze my AWS environment and generate a comprehensive cost optimization report with actionable remediation scripts. Follow the steps below to identify cost anomalies, optimize resource usage, and generate infrastructure-as-code templates for cost savings.

## Prerequisites

Before running this task, ensure you have:

1. **AWS CLI configured** with appropriate credentials
2. **Cost Explorer enabled** in your AWS account (Billing Console → Cost Explorer → Enable)
3. **IAM permissions** for read-only access to:
   - Cost Explorer (`ce:GetCostAndUsage`, `ce:GetCostForecast`)
   - CloudWatch (`cloudwatch:GetMetricStatistics`)
   - EC2 (`ec2:DescribeInstances`, `ec2:DescribeVolumes`, `ec2:DescribeSnapshots`)
   - S3 (`s3:ListAllMyBuckets`, `s3:GetBucketLocation`)
   - RDS (`rds:DescribeDBInstances`, `rds:DescribeDBSnapshots`)
   - Lambda (`lambda:ListFunctions`)
   - EKS (`eks:ListClusters`, `eks:ListNodegroups`)
   - IAM (`iam:ListRoles`, `iam:ListPolicies`) — for permission analysis
4. **CloudWatch Agent installed** (optional, for memory/disk metrics)

## Step 1: Cost Anomaly Detection

Analyze my AWS costs for the last 30 days and identify:

1. **Sudden spikes** (>20% day-over-day increase)
2. **Gradual drift** (>30% month-over-month increase)
3. **Outlier services** (consuming >40% of total bill)
4. **Idle resources** (<5% utilization over 7 days)

Use this command to get cost data:
```bash
aws ce get-cost-and-usage \
  --time-period Start=$(date -u -d '30 days ago' +%Y-%m-%d),End=$(date -u +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE
```

**Output format:**
```markdown
## Cost Anomaly Report

| Metric | Current | Previous | Change | Status |
|--------|---------|----------|--------|--------|
| Total Cost | $X | $Y | +Z% | ⚠️/✅ |
| Top Service | Service A | — | $X | +Y% |

### Anomalies Detected
1. [Anomaly description with evidence]
2. [Anomaly description with evidence]
```

## Step 2: Service-by-Service Analysis

For each of the following services, analyze usage patterns and identify optimization opportunities:

### EC2 Analysis
```bash
# List instances with utilization
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,State.Name,LaunchTime]'

# Get CPU utilization for each instance
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=INSTANCE_ID \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average
```

**Check for:**
- Idle instances (CPU <5% for 7+ days)
- Over-provisioned instances (right-size by 1-2 types)
- Unattached EBS volumes
- Old snapshots (>90 days)

### S3 Analysis
```bash
# List buckets with sizes
aws s3api list-buckets --query 'Buckets[].Name' --output text | \
  xargs -I {} sh -c 'echo -n "{}: "; aws s3 ls s3://{} --recursive --summarize 2>/dev/null | tail -1'
```

**Check for:**
- Wrong storage class (Standard for infrequent access)
- Incomplete multipart uploads
- Old versions without lifecycle policies

### RDS Analysis
```bash
# List instances
aws rds describe-db-instances \
  --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceClass,Engine,AllocatedStorage]'
```

**Check for:**
- Over-provisioned instances (CPU/memory <20%)
- Idle databases (0 connections for 7+ days)
- Excessive backup retention

### Lambda Analysis
```bash
# List functions
aws lambda list-functions --query 'Functions[].[FunctionName,MemorySize,LastModified]'
```

**Check for:**
- Over-provisioned memory
- Unused functions (0 invocations in 30+ days)
- Large deployment packages

### EKS Analysis
```bash
# List clusters and node groups
aws eks list-clusters
aws eks list-nodegroups --cluster-name CLUSTER_NAME
```

**Check for:**
- Over-provisioned nodes
- Idle namespaces
- Unbound PVCs

**Output format:**
```markdown
## Service Analysis

### EC2
| Resource | Issue | Current | Recommended | Est. Savings |
|----------|-------|---------|-------------|--------------|
| i-0abc123 | Idle | m5.large | Stop | $70/month |

### S3
| Bucket | Issue | Current | Recommended | Est. Savings |
|--------|-------|---------|-------------|--------------|
| app-logs | Wrong class | Standard | Standard-IA | $40/month |

[Continue for each service...]
```

## Step 3: Generate Optimization Plan

Based on the analysis, generate a prioritized optimization plan:

```markdown
## Optimization Plan

### Priority 1: Quick Wins (Low Risk, High Impact)
1. [Action] — Est. savings: $X/month — Risk: Low
2. [Action] — Est. savings: $X/month — Risk: Low

### Priority 2: Medium Impact (May require downtime)
1. [Action] — Est. savings: $X/month — Risk: Medium
2. [Action] — Est. savings: $X/month — Risk: Medium

### Priority 3: Long-term (Reserved Instances, Savings Plans)
1. [Action] — Est. savings: $X/month — Risk: Low

**Total Estimated Savings: $X/month ($Y/year)**
```

## Step 4: Generate Remediation Scripts

For each optimization, generate one of:

### Option A: AWS CLI Script (Quick)
```bash
#!/bin/bash
# AWS Cost Guardian — Remediation Script
# Generated: [Date]
# Total Estimated Savings: $X/month
# Risk Level: Low/Medium/High

# === [Action 1] ===
aws [command]
# This does [explanation]

# === Rollback ===
# aws [rollback-command]
```

### Option B: CloudFormation Template (Production)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — Cost Optimization Resources'

Resources:
  # Example: S3 Lifecycle Policy
  S3LifecyclePolicy:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Ref BucketName
      LifecycleConfiguration:
        Rules:
          - Id: MoveToIA
            Status: Enabled
            Transitions:
              - TransitionInDays: 30
                StorageClass: STANDARD_IA
            ExpirationInDays: 90
```

### Option C: Terraform Module (Infrastructure)
```hcl
# AWS Cost Guardian — Cost Optimization Terraform Module

resource "aws_s3_bucket_lifecycle_configuration" "cost_optimization" {
  bucket = aws_s3_bucket.main.id

  rule {
    id     = "move-to-ia"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    expiration {
      days = 90
    }
  }
}
```

**Always provide all three options** so the user can choose their preferred method.

## Step 5: Generate Monitoring Setup

Create CloudWatch alarms for ongoing cost monitoring:

```bash
# Create billing alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "AWS-Cost-Alert" \
  --alarm-description "Alert when AWS costs exceed threshold" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 1000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions SNS_TOPIC_ARN
```

**Output format:**
```markdown
## Monitoring Setup

### Billing Alarms
- [Alarm name]: Alert when costs exceed $X — [CLI command]

### Resource Alarms
- [Alarm name]: Alert when [resource] exceeds [threshold] — [CLI command]
```

## Step 6: Summary Report

Generate a final summary:

```markdown
## Cost Optimization Summary

### Analysis Period
- Start: [Date]
- End: [Date]
- Total Cost Analyzed: $X

### Findings
- [N] anomalies detected
- [N] optimization opportunities
- [N] idle resources identified

### Recommendations
| Priority | Action | Est. Savings | Risk | Implementation |
|----------|--------|--------------|------|----------------|
| 1 | [Action] | $X/month | Low | CLI/CF/TF |
| 2 | [Action] | $X/month | Medium | CLI/CF/TF |

### Total Potential Savings
- Monthly: $X
- Annual: $Y
- Percentage: Z%

### Next Steps
1. Review and approve Priority 1 actions
2. Schedule Priority 2 actions during maintenance window
3. Evaluate Reserved Instance purchases for Priority 3

### Files Generated
- `cost-optimization-report.md` — Full analysis report
- `remediation-cli.sh` — AWS CLI remediation script
- `remediation-cloudformation.yaml` — CloudFormation template
- `remediation-terraform.tf` — Terraform module
- `monitoring-alarms.sh` — CloudWatch alarm setup
```

## Well-Architected Framework Alignment

This task aligns with the AWS Well-Architected Framework:

- **Cost Optimization**: Core focus — identify and eliminate waste
- **Security**: IAM least privilege for remediation, no credential exposure
- **Operational Excellence**: Monitoring setup, automated reporting
- **Reliability**: Right-sizing ensures adequate capacity
- **Performance Efficiency**: Right-sizing ensures optimal performance
- **Sustainability**: Resource efficiency reduces carbon footprint

## Important Notes

1. **Never auto-execute destructive actions** — All scripts are presented for review
2. **Provide rollback commands** — Every action has an undo option
3. **Estimate cost savings** — Every recommendation includes expected savings
4. **Flag risk levels** — Low/Medium/High for each action
5. **Use business context** — Consider weekends, holidays, deployments when analyzing
