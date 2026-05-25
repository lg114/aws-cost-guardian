# Example: Remediation Script Generation

## User Message

"I have 5 idle EC2 instances that I want to clean up. Can you generate a script?"

## Guardian Response

## Remediation: Idle EC2 Instances

**Issue**: 5 EC2 instances have been idle (<2% CPU) for 14+ days
**Estimated Savings**: $350/month
**Risk Level**: Low (stop) / High (terminate)

### Instance Details

| Instance ID | Type | CPU (14-day avg) | Monthly Cost |
|-------------|------|------------------|--------------|
| i-0abc123 | m5.large | 1.2% | $70 |
| i-0def456 | m5.large | 0.8% | $70 |
| i-0ghi789 | m5.xlarge | 1.5% | $140 |
| i-0jkl012 | t3.medium | 0.5% | $30 |
| i-0mno345 | t3.medium | 1.1% | $30 |

### Option 1: Stop Instances (Recommended)

Stopping instances preserves their EBS volumes and allows restart if needed.

```bash
#!/bin/bash
# AWS Cost Guardian — Stop Idle EC2 Instances
# Generated: 2024-05-25
# Estimated Savings: $350/month
# Risk Level: Low

# Stop all idle instances
aws ec2 stop-instances --instance-ids i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345

echo "Instances stopped. EBS volumes preserved."
echo "To restart: aws ec2 start-instances --instance-ids i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345"
```

### Option 2: Terminate Instances

Terminating instances deletes them permanently. **Create AMI backups first.**

```bash
#!/bin/bash
# AWS Cost Guardian — Terminate Idle EC2 Instances
# Generated: 2024-05-25
# Estimated Savings: $350/month
# Risk Level: High

# Create AMI backups before termination
for i in i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345; do
  aws ec2 create-image \
    --instance-id $i \
    --name "backup-${i}-$(date +%Y%m%d)" \
    --no-reboot
done

echo "AMIs created. Waiting 60 seconds for completion..."
sleep 60

# Terminate instances
aws ec2 terminate-instances --instance-ids i-0abc123 i-0def456 i-0ghi789 i-0jkl012 i-0mno345

echo "Instances terminated. AMIs available for recovery."
```

### Option 3: Schedule Stop/Start

If instances are only needed during business hours:

```bash
#!/bin/bash
# AWS Cost Guardian — Schedule EC2 Instances
# Stop at 7pm, Start at 7am (weekdays only)
# Estimated Savings: ~$210/month (60% reduction)

# Create IAM role for Lambda (if not exists)
# ... (Lambda function code to stop/start instances on schedule)
```

**Would you like me to proceed with any of these options?**
