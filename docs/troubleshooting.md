# AWS Cost Guardian — Troubleshooting

## Common Issues

### "I don't have access to Cost Explorer"

**Problem**: The guardian can't analyze costs because Cost Explorer isn't enabled.

**Solution**:
1. Enable Cost Explorer in the AWS Billing Console
2. Wait 24 hours for data to populate
3. Retry the analysis

### "The guardian can't access my AWS resources"

**Problem**: The guardian needs read-only access to analyze resources.

**Solution**:
1. Create an IAM role with read-only access:
   - `ReadOnlyAccess` policy
   - Or custom policy with specific service permissions
2. Provide the role ARN to the guardian
3. Ensure the role has permission to access Cost Explorer

### "Recommendations don't match my actual usage"

**Problem**: The guardian's recommendations seem off.

**Possible Causes**:
1. **Insufficient data**: Wait for more CloudWatch data to accumulate (7-14 days)
2. **Unusual period**: If you analyzed during a spike/deploy, results may be skewed
3. **Missing metrics**: CloudWatch Agent not installed for memory/disk metrics

**Solution**: Provide context about your workload patterns and ask for a re-analysis.

### "The guardian recommended terminating a production instance"

**Problem**: The recommendation seems dangerous.

**Solution**:
1. **Don't panic** — the guardian only recommends, never auto-executes
2. Review the utilization data the guardian provides
3. If it's truly production, mark it with a `Production` tag
4. Ask the guardian to re-analyze with production context

### "Remediation script failed"

**Problem**: The generated script didn't work.

**Common Causes**:
1. **Insufficient IAM permissions**: Your user/role needs write access
2. **Resource changed**: The resource was modified since analysis
3. **Region mismatch**: Script targets wrong region

**Solution**: Check the error message, verify permissions, and ask for an updated script.

### "The guardian doesn't know about [service]"

**Problem**: The guardian seems unfamiliar with a specific AWS service.

**Solution**:
1. The guardian can analyze any AWS service using its general knowledge
2. Provide details about the service and ask for analysis
3. For best results, include CloudWatch metrics and cost data in your question

## Best Practices

1. **Use tags consistently** — Tags enable the guardian to analyze costs by team, project, environment
2. **Enable detailed monitoring** — CloudWatch detailed monitoring provides better data for analysis
3. **Set up billing alerts** — AWS billing alerts complement the guardian's analysis
4. **Review regularly** — Monthly reviews catch issues before they become expensive
5. **Provide context** — Tell the guardian about your workload patterns for better recommendations
