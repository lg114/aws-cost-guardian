# RDS — Cost Analysis Module

## Cost Signals

- Instance class (db.t3.micro, db.r5.large, etc.)
- Storage type and allocated storage
- Read replica count
- Multi-AZ deployment
- Backup retention period
- Connection count and CPU utilization

## Common Waste Patterns

1. **Over-provisioned Instances** — CPU/memory utilization <20%
2. **Idle Databases** — No connections for 7+ days
3. **Excessive Backups** — Retention period longer than needed
4. **Unnecessary Read Replicas** — Replicas with no read traffic
5. **Old Snapshots** — Manual snapshots older than retention policy

## Optimization Playbook

### Right-sizing
- Analyze CloudWatch metrics for 14+ days
- Recommend instance class with 30% headroom
- Consider Aurora Serverless for variable workloads

### Reserved Instances
- For production databases with steady workloads
- Recommend 1-year Standard RI for predictable loads

### Storage Optimization
- Use General Purpose SSD (gp3) for most workloads
- Provisioned IOPS only for high-performance requirements

## Remediation Commands

```bash
# List RDS instances with metrics
aws rds describe-db-instances \
  --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceClass,Engine,AllocatedStorage]'

# Get CPU utilization for an instance
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name CPUUtilization \
  --dimensions Name=DBInstanceIdentifier,Value=INSTANCE_ID \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average

# Stop idle RDS instance (can be restarted)
aws rds stop-db-instance --db-instance-identifier INSTANCE_ID

# Delete old manual snapshots
aws rds describe-db-snapshots --snapshot-type manual \
  --query 'DBSnapshots[?SnapshotCreateTime<=`2024-01-01`].DBSnapshotIdentifier' \
  --output text | \
  xargs -I {} aws rds delete-db-snapshot --db-snapshot-identifier {}
```
