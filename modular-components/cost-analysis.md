# AWS Cost Guardian — Analysis Framework

## Anomaly Detection Patterns

### Sudden Spikes
- **Threshold**: >20% increase day-over-day
- **Detection**: Compare current day's cost to 7-day rolling average
- **Action**: Alert user, identify the service causing the spike

### Gradual Drift
- **Threshold**: >30% increase month-over-month
- **Detection**: Compare current month's running total to same point in previous month
- **Action**: Flag for review, identify trending services

### Outlier Services
- **Detection**: Service consuming >40% of total bill when historical average is <20%
- **Action**: Deep dive into the service's resource utilization

### Unused/Idle Resources
- **Detection**: Resources with <5% utilization over 7 days
- **Action**: Recommend termination or downsizing

## Business Context Awareness

### Time-Based Patterns
- Weekends and holidays should show lower compute costs for most workloads
- Month-end may show higher costs for batch processing
- Quarter-end may show higher costs for reporting/analytics

### Seasonal Trends
- E-commerce: expect spikes during sales events (Black Friday, Prime Day)
- Gaming: expect spikes during releases and weekends
- SaaS: expect steady growth, spikes during onboarding periods

### Deployment Awareness
- Cost spikes immediately after deployments are likely legitimate
- Cost spikes unrelated to deployments warrant investigation

## Cost Dimensions

Analyze costs by:
- **Service**: Which AWS service is driving costs?
- **Region**: Are resources in the optimal region?
- **Tag**: Which team/project/environment is responsible?
- **Account**: Which AWS account in the organization?

## Time Windows

- **Current vs Previous Period**: Day-over-day, week-over-week, month-over-month
- **Rolling Averages**: 7-day, 30-day rolling averages for trend detection
- **Year-over-Year**: For seasonal businesses

## Analysis Output Format

When presenting analysis results, use this structure:

```markdown
## Cost Analysis Summary

| Metric | Current | Previous | Change |
|--------|---------|----------|--------|
| Total Cost | $X | $Y | +Z% |
| Top Service | Service A | — | $X (+Y%) |

### Findings
1. [Finding with evidence]
2. [Finding with evidence]

### Recommendations
1. [Recommendation] — Est. savings: $X/month
2. [Recommendation] — Est. savings: $Y/month
```
