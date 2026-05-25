# Lambda — Cost Analysis Module

## Cost Signals

- Memory allocation (128MB to 10GB)
- Execution duration
- Invocation count
- Cold start frequency
- Concurrent executions

## Common Waste Patterns

1. **Over-provisioned Memory** — Memory set higher than needed
2. **Unused Functions** — Functions with 0 invocations in 30+ days
3. **Excessive Timeouts** — Functions timing out and retrying
4. **Large Deployment Packages** — Packages over 50MB
5. **No Provisioned Concurrency** — High cold start costs for latency-sensitive functions

## Optimization Playbook

### Memory Right-sizing
- Use AWS Lambda Power Tuning to find optimal memory setting
- Start with 128MB, increase until duration stabilizes
- Higher memory = faster execution = lower cost (up to a point)

### Cold Start Optimization
- Use Provisioned Concurrency for latency-sensitive functions
- Minimize package size
- Use Lambda Layers for shared dependencies

### Cleanup
- Delete unused functions
- Archive old versions
- Remove unused Lambda Layers

## Remediation Commands

```bash
# List functions with invocations
aws lambda list-functions --query 'Functions[].[FunctionName,MemorySize,LastModified]'

# Get invocation count for a function
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=FUNCTION_NAME \
  --start-time $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 2592000 \
  --statistics Sum

# Update function memory
aws lambda update-function-configuration \
  --function-name FUNCTION_NAME \
  --memory-size 256

# Delete unused function
aws lambda delete-function --function-name FUNCTION_NAME
```
