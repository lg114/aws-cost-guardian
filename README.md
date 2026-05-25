# AWS Cost Guardian — Cost Optimization Prompt

> **AWS Prompt the Planet Challenge Submission**
>
> An autonomous AWS cost analysis and optimization prompt that identifies cost anomalies, generates remediation scripts, and produces infrastructure-as-code templates for immediate deployment.

## Quick Start

1. Copy the contents of [`PROMPT.md`](PROMPT.md)
2. Paste into your AI assistant (Claude, GPT, Gemini, etc.)
3. The AI will analyze your AWS environment and generate optimization reports

## Prerequisites

Before using this prompt, ensure you have:

### AWS Account Setup
1. **Active AWS account** with billing enabled
2. **Cost Explorer enabled** — Go to Billing Console → Cost Explorer → Enable (wait 24 hours for data)
3. **AWS CLI installed** — Version 2.x recommended
4. **AWS CLI configured** — Run `aws configure` with your credentials

### Required IAM Permissions
Create an IAM policy with these permissions:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CostExplorerRead",
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage",
        "ce:GetCostForecast",
        "ce:GetDimensionValues"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchRead",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:ListMetrics"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EC2Read",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots",
        "ec2:DescribeInstanceTypes"
      ],
      "Resource": "*"
    },
    {
      "Sid": "S3Read",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "*"
    },
    {
      "Sid": "RDSRead",
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "rds:DescribeDBSnapshots"
      ],
      "Resource": "*"
    },
    {
      "Sid": "LambdaRead",
      "Effect": "Allow",
      "Action": [
        "lambda:ListFunctions",
        "lambda:GetFunction"
      ],
      "Resource": "*"
    },
    {
      "Sid": "EKSRead",
      "Effect": "Allow",
      "Action": [
        "eks:ListClusters",
        "eks:ListNodegroups",
        "eks:DescribeCluster"
      ],
      "Resource": "*"
    }
  ]
}
```

### Optional (for enhanced analysis)
- **CloudWatch Agent** — For memory and disk metrics
- **AWS Organizations access** — For multi-account analysis
- **Tagging strategy** — Resources tagged with Environment, Team, Project

## Use Case

### Who Is This For?
- **DevOps Engineers** managing AWS infrastructure
- **Cloud Architects** optimizing resource costs
- **Startup CTOs** looking to reduce cloud spend
- **Enterprise Teams** implementing FinOps practices

### What Problem Does This Solve?
- Unexpected AWS cost spikes going undetected
- Manual cost analysis taking hours of work
- Missing optimization opportunities across services
- Lack of automated remediation scripts

### When to Use This
- Monthly cost reviews
- After detecting billing alerts
- Before Reserved Instance purchases
- During infrastructure audits
- When onboarding new team members

## Expected Outcome

After running this prompt, you will have:

1. **Cost Anomaly Report** — Identified spikes, drift, and outliers
2. **Service-by-Service Analysis** — EC2, S3, RDS, Lambda, EKS breakdown
3. **Optimization Plan** — Prioritized recommendations with savings estimates
4. **Remediation Scripts** — Three formats:
   - AWS CLI scripts (quick execution)
   - CloudFormation templates (production deployment)
   - Terraform modules (infrastructure-as-code)
5. **Monitoring Setup** — CloudWatch alarms for ongoing cost tracking
6. **Summary Report** — Executive overview with total savings potential

## AWS Services Used

| Service | Purpose |
|---------|---------|
| **AWS Cost Explorer** | Cost data retrieval and analysis |
| **Amazon CloudWatch** | Resource utilization metrics |
| **Amazon EC2** | Instance right-sizing, idle detection |
| **Amazon S3** | Storage class optimization, lifecycle policies |
| **Amazon RDS** | Database right-sizing, idle detection |
| **AWS Lambda** | Function optimization, unused detection |
| **Amazon EKS** | Node right-sizing, namespace cleanup |
| **AWS IAM** | Permission analysis, least privilege |
| **AWS CloudFormation** | Infrastructure-as-code output |
| **Amazon SNS** | Billing alert notifications |

## AWS Well-Architected Framework Alignment

### Cost Optimization ✅ (Core Focus)
- Identify and eliminate waste
- Right-size resources based on actual usage
- Implement Reserved Instances and Savings Plans
- Automate cost monitoring

### Security
- IAM least privilege for remediation actions
- No credential exposure in scripts
- Audit trail for all changes
- Generate IAM policies for remediation

### Operational Excellence
- Automated cost reporting
- Monitoring setup for ongoing tracking
- Standardized remediation procedures
- Infrastructure-as-code for version control

### Reliability
- Right-sizing ensures adequate capacity
- Backup verification before cleanup
- Rollback commands for all changes
- CloudFormation/Terraform for rollback support

### Performance Efficiency
- Right-sizing ensures optimal performance
- Resource utilization analysis
- Cold start optimization for Lambda
- Storage tier optimization

### Sustainability
- Resource efficiency reduces carbon footprint
- Eliminate idle resources
- Optimize storage tiers
- Right-size to reduce over-provisioning

## Project Structure

```
├── PROMPT.md                    # Main task prompt (copy this)
├── modular-components/          # Modular design components
│   ├── core-identity.md         # Task execution rules
│   ├── cost-analysis.md         # Analysis framework
│   ├── remediation.md           # Remediation engine
│   └── services/                # Per-service analysis modules
│       ├── ec2.md
│       ├── s3.md
│       ├── rds.md
│       ├── lambda.md
│       └── eks.md
├── examples/                    # Example conversations
├── tools/
│   └── aws-cli-reference.md     # AWS CLI commands reference
└── docs/
    ├── use-cases.md             # Use case scenarios
    └── troubleshooting.md       # Common issues
```

## Troubleshooting

### "Cost Explorer data not available"
**Problem:** Cost Explorer shows no data or errors.
**Solution:**
1. Enable Cost Explorer in Billing Console
2. Wait 24 hours for data to populate
3. Verify IAM permissions include `ce:GetCostAndUsage`

### "CloudWatch metrics missing"
**Problem:** No CPU/memory metrics for resources.
**Solution:**
1. Verify CloudWatch Agent is installed (for memory metrics)
2. Check that detailed monitoring is enabled for EC2
3. Ensure IAM permissions include `cloudwatch:GetMetricStatistics`

### "Remediation script failed"
**Problem:** Generated script returns errors.
**Solution:**
1. Check IAM permissions for write access
2. Verify resource hasn't changed since analysis
3. Ensure correct AWS region is configured
4. Check if resource is in a different account

### "Wrong resource IDs"
**Problem:** Script uses placeholder IDs instead of real ones.
**Solution:**
1. Re-run the prompt and explicitly ask for actual resource IDs
2. Provide your resource IDs in the prompt

### "CloudFormation stack creation failed"
**Problem:** CloudFormation template fails to deploy.
**Solution:**
1. Check CloudFormation events for error details
2. Verify IAM permissions for resource creation
3. Ensure resource names don't conflict with existing resources

## Example Usage

### Basic Cost Analysis
```
User: Analyze my AWS costs for the last 30 days and identify optimization opportunities.

AI: [Generates cost anomaly report, service analysis, optimization plan, and remediation scripts]
```

### Specific Service Focus
```
User: Focus on my EC2 costs. Are my instances the right size?

AI: [Analyzes EC2 instances, provides right-sizing recommendations, generates migration scripts]
```

### Storage Optimization
```
User: My S3 costs are increasing. How can I optimize my storage?

AI: [Analyzes S3 buckets, recommends lifecycle policies, generates CloudFormation template]
```

## Safety Features

- **Never auto-executes** — All scripts presented for review
- **Destructive actions require confirmation** — Always asks before termination/deletion
- **Provides rollback commands** — Shows how to undo changes
- **Estimates cost savings** — Every recommendation includes expected savings
- **Risk level indicators** — Low/Medium/High for each action

## License

MIT License — see [LICENSE](LICENSE)

---

**Built for AWS Prompt the Planet Challenge**
