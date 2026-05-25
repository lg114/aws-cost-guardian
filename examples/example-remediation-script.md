# Example: Remediation Script Generation

## User Message

"I have 5 idle EC2 instances. Generate remediation scripts in all three formats."

## AI Response

## Step 1: Analysis

| Instance ID | Type | CPU (14-day avg) | Status | Monthly Cost |
|-------------|------|------------------|--------|--------------|
| i-0abc123 | m5.large | 1.2% | Idle | $70 |
| i-0def456 | m5.large | 0.8% | Idle | $70 |
| i-0ghi789 | m5.xlarge | 1.5% | Idle | $140 |
| i-0jkl012 | t3.medium | 0.5% | Idle | $30 |
| i-0mno345 | t3.medium | 1.1% | Idle | $30 |

**Total Monthly Cost: $340**
**Estimated Savings: $340/month ($4,080/year)**

## Step 2: Remediation Scripts

### Option A: AWS CLI (Quick)
```bash
#!/bin/bash
# AWS Cost Guardian — Stop Idle EC2 Instances
# Generated: 2024-05-25
# Estimated Savings: $340/month
# Risk Level: Low

# Stop all idle instances
aws ec2 stop-instances --instance-ids i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345

echo "Instances stopped. EBS volumes preserved."
echo "To restart: aws ec2 start-instances --instance-ids i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345"
```

### Option B: CloudFormation (Production)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS Cost Guardian — Stop Idle EC2 Instances'

Resources:
  StopIdleInstancesPolicy:
    Type: 'AWS::IAM::Policy'
    Properties:
      PolicyName: 'StopIdleEC2Instances'
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Action: 'ec2:StopInstances'
            Resource:
              - 'arn:aws:ec2:*:*:instance/i-0abc123'
              - 'arn:aws:ec2:*:*:instance/i-0def456'
              - 'arn:aws:ec2:*:*:instance/i-0ghi789'
              - 'arn:aws:ec2:*:*:instance/i-0jkl012'
              - 'arn:aws:ec2:*:*:instance/i-0mno345'
```

### Option C: Terraform (Infrastructure)
```hcl
# AWS Cost Guardian — Stop Idle EC2 Instances
# Generated: 2024-05-25
# Estimated Savings: $340/month

variable "idle_instances" {
  description = "List of idle EC2 instance IDs"
  type        = list(string)
  default     = ["i-0abc123", "i-0def456", "i-0ghi789", "i-0jkl012", "i-0mno345"]
}

resource "aws_ec2_instance_state" "idle" {
  for_each    = toset(var.idle_instances)
  instance_id = each.value
  state       = "stopped"
}

output "stopped_instances" {
  description = "List of stopped instance IDs"
  value       = values(aws_ec2_instance_state.idle)[*].instance_id
}
```

## Step 3: Monitoring Setup

```bash
# Create alarm for idle instances
aws cloudwatch put-metric-alarm \
  --alarm-name "Idle-EC2-Alert" \
  --alarm-description "Alert when EC2 CPU < 5% for 7 days" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 604800 \
  --threshold 5 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:idle-alerts
```
