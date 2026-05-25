# AWS Cost Guardian — Cost Optimization Task

## Task

Analyze my AWS environment and generate a comprehensive cost optimization report with actionable remediation scripts. Follow the steps below to identify cost anomalies, optimize resource usage, and generate infrastructure-as-code templates for cost savings.

## Prerequisites

Before running this task, ensure you have:

### AWS Account Setup
1. **Active AWS account** with billing enabled
2. **Cost Explorer enabled** — Go to Billing Console → Cost Explorer → Enable (wait 24 hours for data)
3. **AWS CLI installed** — Version 2.x recommended
4. **AWS CLI configured** — Run `aws configure` with your credentials

### Required IAM Permissions
Create an IAM policy with these permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CostExplorerRead",
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage",
        "ce:GetCostForecast",
        "ce:GetDimensionValues"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchRead",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:ListMetrics"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EC2Read",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots",
        "ec2:DescribeInstanceTypes"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3Read",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "*"
    },
    {
      "Sid": "RDSRead",
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "rds:DescribeDBSnapshots"
      ],
      "Resource": "*"
    },
    {
      "Sid": "LambdaRead",
      "Effect": "Allow",
      "Action": [
        "lambda:ListFunctions",
        "lambda:GetFunction"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EKSRead",
      "Effect": "Allow",
      "Action": [
        "eks:ListClusters",
        "eks:ListNodegroups",
        "eks:DescribeCluster"
      ],
      "Resource": "*"
    }
  ]
}
```

### Optional (for enhanced analysis)
- **CloudWatch Agent** — For memory and disk metrics
- **AWS Organizations access** — For multi-account analysis
- **Tagging strategy** — Resources tagged with Environment, Team, Project

## Step 1: Cost Anomaly Detection

Analyze my AWS costs for the last 30 days and identify:

1. **Sudden spikes** (>20% day-over-day increase)
2. **Gradual drift** (>30% month-over-month increase)
3. **Outlier services** (consuming >40% of total bill)
4. **Idle resources** (<5% utilization over 7 days)

**IMPORTANT:** Use actual resource IDs from my environment. Do NOT use placeholders like `INSTANCE_ID`.

Run this command to get cost data:
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

For each of the following services, analyze usage patterns and identify optimization opportunities.

**IMPORTANT:** Use actual resource IDs from my environment. Replace placeholders with real values.

### EC2 Analysis
```bash
# List instances with utilization
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,State.Name,LaunchTime]'

# Get CPU utilization for each instance
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=ACTUAL_INSTANCE_ID \
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
aws eks list-nodegroups --cluster-name ACTUAL_CLUSTER_NAME
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

For each optimization, generate scripts in all three formats. **Use actual resource IDs from my environment.**

### Decision Guide: When to Use Each Format

| Format | Use When | Pros | Cons |
|--------|----------|------|------|
| **AWS CLI** | Quick testing, one-time changes | Fast, immediate | No version control, hard to rollback |
| **CloudFormation** | Production AWS environments | Version controlled, rollback support | AWS-specific, verbose |
| **Terraform** | Multi-cloud or complex infrastructure | Cloud-agnostic, modular | Requires Terraform setup |

### Option A: AWS CLI Script (Quick Testing)
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

### Option B: CloudFormation Template (Production Deployment)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — Cost Optimization Resources'

Parameters:
  Environment:
    Type: String
    Default: production
    AllowedValues: [production, staging, development]

Resources:
  # S3 Lifecycle Policy
  S3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub '${Environment}-optimized-bucket'
      LifecycleConfiguration:
        Rules:
          - Id: MoveToIA
            Status: Enabled
            Transitions:
              - TransitionInDays: 30
                StorageClass: STANDARD_IA
            ExpirationInDays: 90

  # Billing Alarm
  BillingAlarm:
    Type: 'AWS::CloudWatch::Alarm'
    Properties:
      AlarmName: !Sub '${Environment}-billing-alarm'
      AlarmDescription: 'Alert when AWS costs exceed threshold'
      MetricName: EstimatedCharges
      Namespace: AWS/Billing
      Statistic: Maximum
      Period: 21600
      Threshold: 1000
      ComparisonOperator: GreaterThanThreshold
      EvaluationPeriods: 1
      AlarmActions:
        - !Ref SNSTopic

  SNSTopic:
    Type: 'AWS::SNS::Topic'
    Properties:
      TopicName: !Sub '${Environment}-cost-alerts'

Outputs:
  BucketName:
    Value: !Ref S3Bucket
  SNSTopicArn:
    Value: !Ref SNSTopic
```

### Option C: Terraform Module (Infrastructure-as-Code)
```hcl
# AWS Cost Guardian — Cost Optimization Terraform Module

variable "environment" {
  description = "Environment name (production, staging, development)"
  type        = string
  default     = "production"
}

variable "billing_threshold" {
  description = "Billing alarm threshold in USD"
  type        = number
  default     = 1000
}

# S3 Lifecycle Policy
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

# Billing Alarm
resource "aws_cloudwatch_metric_alarm" "billing" {
  alarm_name          = "${var.environment}-billing-alarm"
  alarm_description   = "Alert when AWS costs exceed threshold"
  metric_name         = "EstimatedCharges"
  namespace           = "AWS/Billing"
  statistic           = "Maximum"
  period              = 21600
  threshold           = var.billing_threshold
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  alarm_actions       = [aws_sns_topic.cost_alerts.arn]
}

resource "aws_sns_topic" "cost_alerts" {
  name = "${var.environment}-cost-alerts"
}

output "sns_topic_arn" {
  value = aws_sns_topic.cost_alerts.arn
}
```

## Step 5: Generate Monitoring Setup

Create CloudWatch alarms for ongoing cost monitoring:

```bash
# Create SNS topic for alerts
aws sns create-topic --name cost-alerts

# Create billing alarm
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
  --alarm-actions arn:aws:sns:REGION:ACCOUNT_ID:cost-alerts

# Create EC2 CPU utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-High-CPU" \
  --alarm-description "Alert when EC2 CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --dimensions Name=InstanceId,Value=ACTUAL_INSTANCE_ID \
  --alarm-actions arn:aws:sns:REGION:ACCOUNT_ID:cost-alerts
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

## Troubleshooting

### "Cost Explorer data not available"
**Problem:** Cost Explorer shows no data or errors.
**Solution:**
1. Enable Cost Explorer in Billing Console
2. Wait 24 hours for data to populate
3. Verify IAM permissions include `ce:GetCostAndUsage`

### "CloudWatch metrics missing"
**Problem:** No CPU/memory metrics for resources.
**Solution:**
1. Verify CloudWatch Agent is installed (for memory metrics)
2. Check that detailed monitoring is enabled for EC2
3. Ensure IAM permissions include `cloudwatch:GetMetricStatistics`

### "Remediation script failed"
**Problem:** Generated script returns errors.
**Solution:**
1. Check IAM permissions for write access
2. Verify resource hasn't changed since analysis
3. Ensure correct AWS region is configured
4. Check if resource is in a different account

### "Wrong resource IDs"
**Problem:** Script uses placeholder IDs instead of real ones.
**Solution:**
1. Re-run the prompt and explicitly ask for actual resource IDs
2. Provide your resource IDs in the prompt

### "CloudFormation stack creation failed"
**Problem:** CloudFormation template fails to deploy.
**Solution:**
1. Check CloudFormation events for error details
2. Verify IAM permissions for resource creation
3. Ensure resource names don't conflict with existing resources

## Well-Architected Framework Alignment

This task aligns with the AWS Well-Architected Framework:

### Cost Optimization ✅ (Core Focus)
- Identify and eliminate waste
- Right-size resources based on actual usage
- Implement Reserved Instances and Savings Plans
- Automate cost monitoring

### Security
- IAM least privilege for remediation actions
- No credential exposure in scripts
- Audit trail for all changes
- Generate IAM policies for remediation

### Operational Excellence
- Automated cost reporting
- Monitoring setup for ongoing tracking
- Standardized remediation procedures
- Infrastructure-as-code for version control

### Reliability
- Right-sizing ensures adequate capacity
- Backup verification before cleanup
- Rollback commands for all changes
- CloudFormation/Terraform for rollback support

### Performance Efficiency
- Right-sizing ensures optimal performance
- Resource utilization analysis
- Cold start optimization for Lambda
- Storage tier optimization

### Sustainability
- Resource efficiency reduces carbon footprint
- Eliminate idle resources
- Optimize storage tiers
- Right-size to reduce over-provisioning

## Important Notes

1. **Never auto-execute destructive actions** — All scripts are presented for review
2. **Provide rollback commands** — Every action has an undo option
3. **Estimate cost savings** — Every recommendation includes expected savings
4. **Flag risk levels** — Low/Medium/High for each action
5. **Use actual resource IDs** — Do not use placeholders
6. **Verify before cleanup** — Check for dependencies before deleting resources
