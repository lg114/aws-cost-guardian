# AWS Cost Guardian — Use Cases

## 1. Monthly Cost Review

**Scenario**: You want to review your AWS costs at the end of each month.

**Prompt**: "Analyze my AWS costs for this month. What changed compared to last month?"

**Expected Output**: Cost breakdown by service, month-over-month comparison, top cost drivers, recommendations for next month.

## 2. Cost Spike Investigation

**Scenario**: You received a billing alert and need to investigate.

**Prompt**: "My AWS bill jumped from $X to $Y this month. What happened?"

**Expected Output**: Root cause analysis, specific resources causing the spike, remediation script to address the issue.

## 3. Right-sizing Recommendations

**Scenario**: You suspect your instances are over-provisioned.

**Prompt**: "Are my EC2 instances the right size? Can I save money by right-sizing?"

**Expected Output**: Utilization analysis, right-sizing recommendations, migration script.

## 4. Storage Optimization

**Scenario**: Your S3 costs are growing faster than expected.

**Prompt**: "My S3 costs are increasing. How can I optimize my storage?"

**Expected Output**: Storage class analysis, lifecycle policy recommendations, cleanup script.

## 5. Idle Resource Cleanup

**Scenario**: You want to clean up unused resources.

**Prompt**: "Find and help me clean up idle AWS resources."

**Expected Output**: List of idle resources (EC2, RDS, EBS, etc.), estimated savings, cleanup script.

## 6. Reserved Instance Planning

**Scenario**: You want to evaluate Reserved Instance purchases.

**Prompt**: "Should I buy Reserved Instances? Which ones would save me the most?"

**Expected Output**: RI coverage analysis, purchase recommendations, estimated savings.

## 7. Pre-deployment Cost Estimate

**Scenario**: You're planning a new deployment and want to estimate costs.

**Prompt**: "I'm planning to deploy [description]. What will it cost?"

**Expected Output**: Cost estimate by service, optimization suggestions, comparison to alternatives.

## 8. Tag Compliance Audit

**Scenario**: You want to ensure all resources are properly tagged.

**Prompt**: "Are all my AWS resources properly tagged? Which ones are missing tags?"

**Expected Output**: Tag compliance report, list of untagged resources, tagging script.
