# EC2 — Cost Analysis Module

## Cost Signals

- Instance type and family (e.g., m5.large, c5.xlarge)
- Purchase option: On-Demand vs Reserved vs Spot
- CPU utilization (CloudWatch: `CPUUtilization`)
- Memory utilization (requires CloudWatch Agent)
- Network I/O
- EBS volume size and IOPS

## Common Waste Patterns

1. **Idle Instances** — CPU <5% for 7+ days
2. **Over-provisioned Instances** — Right-sized down by 1-2 instance types
3. **Unattached EBS Volumes** — Volumes not attached to any instance
4. **Old Snapshots** — EBS snapshots older than 90 days with no retention policy
5. **Reserved Instance Underutilization** — RI coverage but unused capacity

## Optimization Playbook

### Right-sizing
- Analyze CloudWatch metrics for 14+ days
- Recommend instance type with 20-30% headroom above peak utilization
- Consider burstable instances (t3/t3a) for variable workloads

### Reserved Instances
- For steady-state workloads (>70% utilization over 30 days)
- Recommend 1-year Standard RI for predictable workloads
- Recommend Convertible RI for workloads that may change

### Spot Instances
- For fault-tolerant, flexible workloads (batch processing, CI/CD)
- Always set up Spot Fleet with multiple instance types

## Remediation Commands

```bash
# List idle instances (CPU <5% for 7 days)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-INSTANCE_ID \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average

# Stop idle instance
aws ec2 stop-instances --instance-ids i-INSTANCE_ID

# List unattached EBS volumes
aws ec2 describe-volumes --filters Name=status,Values=available

# Delete unattached volume (after confirmation)
aws ec2 delete-volume --volume-id vol-VOLUME_ID
```
