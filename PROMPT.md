# AWS Cost Guardian — Cost Optimization Task

## Task

Analyze my AWS environment and generate a cost optimization report with remediation scripts. Use actual resource IDs from my environment — never use placeholders.

## Step 1: Cost Anomaly Detection

Analyze my AWS costs for the last 30 days and identify sudden spikes (>20% day-over-day), gradual drift (>30% month-over-month), outlier services (>40% of bill), and idle resources (<5% utilization for 7 days).

```bash
aws ce get-cost-and-usage \
  --time-period Start=$(date -u -d '30 days ago' +%Y-%m-%d),End=$(date -u +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE
```

**Output:** Table with columns: Metric | Current | Previous | Change | Status

## Step 2: Service Analysis

Analyze EC2, S3, RDS, Lambda, and EKS for optimization opportunities.

**EC2:** Check idle instances (CPU <5% for 7 days), over-provisioned instances, unattached EBS volumes, old snapshots (>90 days)
```bash
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,State.Name,LaunchTime]'
```

**Example findings:**
- i-0abc123 (m5.xlarge, $140/month): CPU 3% — Recommend: Stop or right-size to t3.medium ($30/month)
- i-0def456 (m5.2xlarge, $280/month): CPU 8% — Recommend: Right-size to m5.xlarge ($140/month)
- vol-0ghi789 (100GB EBS, $10/month): Unattached — Recommend: Delete after backup

**S3:** Check wrong storage class, incomplete multipart uploads, old versions without lifecycle policies
```bash
aws s3api list-buckets --query 'Buckets[].Name' --output text | xargs -I {} sh -c 'echo -n "{}: "; aws s3 ls s3://{} --recursive --summarize 2>/dev/null | tail -1'
```

**Example findings:**
- app-logs-prod (5TB Standard, $115/month): Move to Standard-IA ($57.50/month) — Save $57.50/month
- user-uploads (2TB Standard, $46/month): Apply lifecycle policy — Save $23/month
- 1,247 incomplete multipart uploads: Clean up — Save $5/month

**RDS:** Check over-provisioned instances (CPU/memory <20%), idle databases (0 connections for 7 days)
```bash
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceClass,Engine,AllocatedStorage]'
```

**Example findings:**
- prod-db (db.r5.xlarge, $370/month): CPU 12%, 5 connections — Recommend: Right-size to db.r5.large ($185/month)
- staging-db (db.r5.large, $185/month): 0 connections for 14 days — Recommend: Stop or delete

**Lambda:** Check over-provisioned memory, unused functions (0 invocations in 30 days)
```bash
aws lambda list-functions --query 'Functions[].[FunctionName,MemorySize,LastModified]'
```

**Example findings:**
- data-processor (1024MB, 100K invocations/month): Optimal at 512MB — Save $15/month
- old-report-generator (512MB, 0 invocations): Unused for 60 days — Recommend: Delete

**EKS:** Check over-provisioned nodes, idle namespaces, unbound PVCs
```bash
aws eks list-clusters && aws eks list-nodegroups --cluster-name CLUSTER_NAME
```

**Example findings:**
- 3x m5.xlarge nodes ($420/month): CPU 25% — Recommend: Right-size to m5.large ($210/month)
- staging namespace: No pods for 30 days — Recommend: Delete namespace
- 2 unbound PVCs (200GB, $20/month): Not attached — Recommend: Delete after verification

**Output:** Table per service with columns: Resource | Issue | Current | Recommended | Est. Savings

## Step 3: Optimization Plan

Generate a prioritized plan with three levels:
1. **Quick Wins** (Low Risk, High Impact) — Safe changes like adding tags, changing storage class
2. **Medium Impact** (May require downtime) — Resizing instances, changing instance types
3. **Long-term** (Reserved Instances, Savings Plans) — Commitment-based savings

**Output:** Table with columns: Priority | Action | Est. Savings | Risk

## Step 4: Remediation Scripts

For each optimization, generate scripts in all three formats. Use actual resource IDs.

**AWS CLI** — Quick testing, one-time changes. Include rollback commands.
```bash
#!/bin/bash
# Stop idle EC2 instances
# Estimated Savings: $420/month (3x m5.xlarge at $140/month each)
# Risk: Low (instances can be restarted)

aws ec2 stop-instances --instance-ids i-0abc123def456 i-0def789ghi012 i-0ghi345jkl678

# Rollback:
# aws ec2 start-instances --instance-ids i-0abc123def456 i-0def789ghi012 i-0ghi345jkl678
```

**CloudFormation** — Production deployment with version control and rollback support.
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'S3 Lifecycle Policy - Move logs to Standard-IA after 30 days'

Parameters:
  BucketName:
    Type: String
    Default: app-logs-prod

Resources:
  S3Bucket:
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
```

**Terraform** — Multi-cloud or complex infrastructure. Include variables and outputs.
```hcl
# S3 Lifecycle Policy - Move logs to Standard-IA after 30 days
resource "aws_s3_bucket_lifecycle_configuration" "logs" {
  bucket = "app-logs-prod"

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

## Step 5: Monitoring Setup

Create CloudWatch alarms for ongoing cost monitoring:
```bash
# Billing alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "AWS-Cost-Alert" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 1000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions SNS_TOPIC_ARN
```

**Output:** List of alarms with names, thresholds, and CLI commands

## Step 6: Summary Report

Generate final summary:
- **Analysis Period:** Start date, End date, Total cost analyzed
- **Findings:** Number of anomalies, optimization opportunities, idle resources
- **Recommendations:** Table with Priority | Action | Est. Savings | Risk | Implementation format
- **Total Savings:** Monthly, Annual, Percentage
- **Next Steps:** Review priorities, schedule maintenance window, evaluate Reserved Instances
- **Files Generated:** List of output files (report, CLI script, CloudFormation template, Terraform module, monitoring setup)

## Well-Architected Framework Alignment

This task aligns with all 6 pillars of the AWS Well-Architected Framework:

- **Cost Optimization**: Identify waste, right-size resources, implement Reserved Instances/Savings Plans
- **Security**: IAM least privilege, no credential exposure, audit trail for changes
- **Operational Excellence**: Automated reporting, monitoring setup, standardized procedures
- **Reliability**: Right-sizing ensures capacity, backup verification, rollback commands
- **Performance Efficiency**: Right-sizing ensures performance, resource utilization analysis
- **Sustainability**: Eliminate idle resources, optimize storage tiers, reduce over-provisioning

## Important Rules

1. **Never auto-execute destructive actions** — Present all scripts for review
2. **Provide rollback commands** — Every action has an undo option
3. **Estimate cost savings** — Every recommendation includes expected savings
4. **Flag risk levels** — Low/Medium/High for each action
5. **Use actual resource IDs** — Never use placeholders
6. **Verify before cleanup** — Check for dependencies before deleting resources
