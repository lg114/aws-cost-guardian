# AWS Cost Guardian — Remediation Engine

## Remediation Flow

```
Detect Issue → Analyze Impact → Generate Scripts → Present for Review → User Executes
```

## Script Generation Rules

1. **Generate three formats** — AWS CLI, CloudFormation, Terraform
2. **Include comments** — explain what each command does
3. **Provide rollback commands** — where applicable, show how to undo the change
4. **Estimate cost savings** — every recommendation must include estimated monthly savings
5. **Flag destructive actions** — clearly mark commands that delete or terminate resources

## Output Formats

### Option A: AWS CLI Script (Quick Execution)

```bash
#!/bin/bash
# AWS Cost Guardian — Remediation Script
# Generated: [Date]
# Issue: [Description]
# Estimated Savings: $X/month
# Risk Level: Low/Medium/High

# === Step 1: [Description] ===
aws [service] [command] [options]
# This command does [explanation]

# === Rollback (if needed) ===
# aws [service] [command] [options]
# This command reverses [explanation]
```

### Option B: CloudFormation Template (Production Deployment)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — [Description]'

Parameters:
  ResourceName:
    Type: String
    Description: Name of the resource to optimize

Resources:
  # Example: S3 Lifecycle Policy
  S3LifecyclePolicy:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Ref ResourceName
      LifecycleConfiguration:
        Rules:
          - Id: MoveToIA
            Status: Enabled
            Transitions:
              - TransitionInDays: 30
                StorageClass: STANDARD_IA
            ExpirationInDays: 90

Outputs:
  ResourceId:
    Description: ID of the optimized resource
    Value: !Ref S3LifecyclePolicy
```

### Option C: Terraform Module (Infrastructure-as-Code)

```hcl
# AWS Cost Guardian — Cost Optimization Terraform Module
# Generated: [Date]
# Estimated Savings: $X/month

variable "resource_name" {
  description = "Name of the resource to optimize"
  type        = string
}

# Example: S3 Lifecycle Policy
resource "aws_s3_bucket_lifecycle_configuration" "cost_optimization" {
  bucket = var.resource_name

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

output "resource_id" {
  description = "ID of the optimized resource"
  value       = aws_s3_bucket_lifecycle_configuration.cost_optimization.id
}
```

## Risk Levels

- **Low**: Can be safely executed (e.g., adding tags, changing storage class)
- **Medium**: May cause brief disruption (e.g., resizing instances, changing instance types)
- **High**: Destructive or irreversible (e.g., terminating instances, deleting buckets)

## Destructive Action Protocol

For High-risk actions:
1. Clearly state what will be destroyed
2. Ask for explicit user confirmation
3. Suggest taking a snapshot/backup first
4. Provide the backup command before the destructive command

## Example Remediation Output

### Issue: Idle EC2 Instance

**Issue**: EC2 instance `i-0abc123def456` has been idle (<2% CPU) for 14 days
**Estimated Savings**: $45/month (m5.large On-Demand)
**Risk Level**: Low (stop) / High (terminate)

### Option 1: AWS CLI (Quick)
```bash
# Stop the idle instance (can be restarted later)
aws ec2 stop-instances --instance-ids i-0abc123def456

# Rollback:
# aws ec2 start-instances --instance-ids i-0abc123def456
```

### Option 2: CloudFormation (Production)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Stop idle EC2 instance'

Resources:
  IdleInstancePolicy:
    Type: 'AWS::IAM::Policy'
    Properties:
      PolicyName: 'StopIdleInstance'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action: 'ec2:StopInstances'
            Resource: 'arn:aws:ec2:*:*:instance/i-0abc123def456'
```

### Option 3: Terraform (Infrastructure)
```hcl
# Stop idle EC2 instance
resource "aws_ec2_instance_state" "idle_instance" {
  instance_id = "i-0abc123def456"
  state       = "stopped"
}

# Rollback: Change state to "running"
```

## Monitoring Setup

After remediation, set up monitoring to prevent future issues:

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
