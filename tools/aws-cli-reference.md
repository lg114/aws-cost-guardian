# AWS CLI Reference — Cost Analysis Commands

## Cost Explorer

```bash
# Get cost and usage for the last 30 days
aws ce get-cost-and-usage \
  --time-period Start=$(date -u -d '30 days ago' +%Y-%m-%d),End=$(date -u +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# Get cost by region
aws ce get-cost-and-usage \
  --time-period Start=$(date -u -d '30 days ago' +%Y-%m-%d),End=$(date -u +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=REGION

# Get cost by tag
aws ce get-cost-and-usage \
  --time-period Start=$(date -u -d '30 days ago' +%Y-%m-%d),End=$(date -u +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=Environment
```

## EC2

```bash
# List instances with details
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].[InstanceId,InstanceType,State.Name,LaunchTime]'

# Get CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-INSTANCE_ID \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average

# List unattached EBS volumes
aws ec2 describe-volumes --filters Name=status,Values=available

# List snapshots older than 90 days
aws ec2 describe-snapshots --owner-ids self \
  --query 'Snapshots[?StartTime<=`2024-01-01`].[SnapshotId,StartTime,VolumeSize]'
```

## S3

```bash
# List buckets with sizes
aws s3api list-buckets --query 'Buckets[].Name' --output text | \
  xargs -I {} sh -c 'echo -n "{}: "; aws s3 ls s3://{} --recursive --summarize 2>/dev/null | tail -1'

# List incomplete multipart uploads
aws s3api list-multipart-uploads --bucket BUCKET_NAME

# Get bucket size (CloudWatch)
aws cloudwatch get-metric-statistics \
  --namespace AWS/S3 \
  --metric-name BucketSizeBytes \
  --dimensions Name=BucketName,Value=BUCKET_NAME,Name=StorageType,Value=StandardStorage \
  --start-time $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average
```

## RDS

```bash
# List instances
aws rds describe-db-instances \
  --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceClass,Engine,AllocatedStorage]'

# Get CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name CPUUtilization \
  --dimensions Name=DBInstanceIdentifier,Value=INSTANCE_ID \
  --start-time $(date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Average

# List manual snapshots
aws rds describe-db-snapshots --snapshot-type manual
```

## Lambda

```bash
# List functions
aws lambda list-functions --query 'Functions[].[FunctionName,MemorySize,LastModified]'

# Get invocation count
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=FUNCTION_NAME \
  --start-time $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 2592000 \
  --statistics Sum

# Get function duration
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=FUNCTION_NAME \
  --start-time $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 2592000 \
  --statistics Average
```

## EKS

```bash
# List clusters
aws eks list-clusters

# List node groups
aws eks list-nodegroups --cluster-name CLUSTER_NAME

# Get node group details
aws eks describe-nodegroup \
  --cluster-name CLUSTER_NAME \
  --nodegroup-name NODEGROUP_NAME
```
