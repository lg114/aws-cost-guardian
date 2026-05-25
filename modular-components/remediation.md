# AWS Cost Guardian — Remediation Engine

## Remediation Flow

```
Detect Issue → Analyze Impact → Generate Script → Present for Review → User Executes
```

## Script Generation Rules

1. **Always generate AWS CLI commands** — not SDK code, not console instructions
2. **Include comments** — explain what each command does
3. **Provide rollback commands** — where applicable, show how to undo the change
4. **Estimate cost savings** — every recommendation must include estimated monthly savings
5. **Flag destructive actions** — clearly mark commands that delete or terminate resources

## Script Template

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

```markdown
## Remediation: Idle EC2 Instance

**Issue**: EC2 instance `i-0abc123def456` has been idle (<2% CPU) for 14 days
**Estimated Savings**: $45/month (m5.large On-Demand)
**Risk Level**: Low (stop) / High (terminate)

### Option 1: Stop Instance (Recommended)
```bash
# Stop the idle instance (can be restarted later)
aws ec2 stop-instances --instance-ids i-0abc123def456
```

### Option 2: Terminate Instance
```bash
# WARNING: This permanently deletes the instance
# Create an AMI backup first:
aws ec2 create-image --instance-id i-0abc123def456 --name "backup-before-terminate-$(date +%Y%m%d)"

# Then terminate:
aws ec2 terminate-instances --instance-ids i-0abc123def456
```
```
