# Example: Cost Optimization

## User Message

"I want to optimize my AWS costs. Can you analyze my environment and suggest savings?"

## Guardian Response

## Cost Analysis Summary

| Service | Current Cost | Optimized Cost | Savings |
|---------|--------------|----------------|---------|
| EC2 | $2,400/month | $1,680/month | $720 (30%) |
| S3 | $300/month | $180/month | $120 (40%) |
| RDS | $600/month | $420/month | $180 (30%) |
| Lambda | $50/month | $40/month | $10 (20%) |
| **Total** | **$3,350/month** | **$2,320/month** | **$1,030 (31%)** |

### EC2 Optimization ($720/month savings)

**Current State:**
- 10x m5.xlarge instances running 24/7
- Average CPU utilization: 25%
- All On-Demand pricing

**Recommendations:**
1. Right-size to m5.large (sufficient for 25% utilization) — Save $360/month
2. Purchase 1-year Reserved Instances for steady-state — Save $360/month

**Action:**
```bash
# Right-size instances (stop, change type, start)
for i in i-001 i-002 i-003 i-004 i-005 i-006 i-007 i-008 i-009 i-010; do
  aws ec2 stop-instances --instance-ids $i
  aws ec2 modify-instance-attribute --instance-id $i --instance-type m5.large
  aws ec2 start-instances --instance-ids $i
done
```

### S3 Optimization ($120/month savings)

**Current State:**
- 5TB in Standard storage
- No lifecycle policies
- 500GB of logs older than 30 days

**Recommendations:**
1. Move logs to Standard-IA after 30 days — Save $60/month
2. Expire old logs after 90 days — Save $60/month

**Action:**
```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-app-logs \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "OptimizeLogs",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}],
      "Expiration": {"Days": 90}
    }]
  }'
```

### RDS Optimization ($180/month savings)

**Current State:**
- 2x db.r5.xlarge instances
- Average CPU: 15%
- No Reserved Instances

**Recommendations:**
1. Right-size to db.r5.large — Save $90/month
2. Purchase 1-year RI — Save $90/month

**Action:**
```bash
# Modify instance class (requires downtime)
aws rds modify-db-instance \
  --db-instance-identifier prod-db \
  --db-instance-class db.r5.large \
  --apply-immediately
```

### Lambda Optimization ($10/month savings)

**Current State:**
- 50 functions, all at 512MB memory
- Power Tuning shows optimal at 256MB for 40 functions

**Recommendations:**
1. Right-size memory for 40 functions — Save $10/month

**Action:**
```bash
# Update function memory
aws lambda update-function-configuration \
  --function-name my-function \
  --memory-size 256
```
