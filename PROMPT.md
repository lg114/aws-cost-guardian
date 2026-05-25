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

**S3:** Check wrong storage class, incomplete multipart uploads, old versions without lifecycle policies
```bash
aws s3api list-buckets --query 'Buckets[].Name' --output text | xargs -I {} sh -c 'echo -n "{}: "; aws s3 ls s3://{} --recursive --summarize 2>/dev/null | tail -1'
```

**RDS:** Check over-provisioned instances (CPU/memory <20%), idle databases (0 connections for 7 days)
```bash
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceClass,Engine,AllocatedStorage]'
```

**Lambda:** Check over-provisioned memory, unused functions (0 invocations in 30 days)
```bash
aws lambda list-functions --query 'Functions[].[FunctionName,MemorySize,LastModified]'
```

**EKS:** Check over-provisioned nodes, idle namespaces, unbound PVCs
```bash
aws eks list-clusters && aws eks list-nodegroups --cluster-name CLUSTER_NAME
```

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
# Estimated Savings: $X/month | Risk: Low/Medium/High
aws [command]
# Rollback: aws [rollback-command]
```

**CloudFormation** — Production deployment with version control and rollback support.
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Cost Optimization'
Parameters:
  Environment:
    Type: String
    Default: production
Resources:
  # Resource definitions
```

**Terraform** — Multi-cloud or complex infrastructure. Include variables and outputs.
```hcl
variable "environment" {
  type    = string
  default = "production"
}
# Resource definitions
output "resource_id" {
  value = resource.name.id
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

## Important Rules

1. **Never auto-execute destructive actions** — Present all scripts for review
2. **Provide rollback commands** — Every action has an undo option
3. **Estimate cost savings** — Every recommendation includes expected savings
4. **Flag risk levels** — Low/Medium/High for each action
5. **Use actual resource IDs** — Never use placeholders
6. **Verify before cleanup** — Check for dependencies before deleting resources
