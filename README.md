# AWS Cost Guardian

An autonomous AI guardian system prompt for protecting AWS environments from unexpected cloud cost spikes, billing anomalies, and resource abuse through intelligent analysis and semi-automated remediation.

## Quick Start

1. Copy the contents of [`PROMPT.md`](PROMPT.md)
2. Paste into your AI assistant's system prompt (Claude, GPT, Gemini, etc.)
3. Ask the guardian to analyze your AWS costs

## What It Does

The AWS Cost Guardian can:

- **Detect cost anomalies** — Identify sudden spikes, gradual drift, and outlier services
- **Analyze by service** — Deep dive into EC2, S3, RDS, Lambda, EKS costs
- **Generate remediation scripts** — AWS CLI commands to fix issues (you review and run)
- **Estimate savings** — Every recommendation includes estimated monthly savings

## Example Usage

```
User: My AWS bill jumped from $500 to $1,200 this month. What happened?

Guardian: [Analyzes costs, identifies root cause, generates remediation script]
```

See more examples in the [`examples/`](examples/) directory.

## Project Structure

```
├── PROMPT.md                    # Main system prompt (copy this)
├── modular-components/          # Modular design components
│   ├── core-identity.md         # Guardian persona and guardrails
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

## Customization

The guardian supports any AWS service. For services not in the default modules, simply ask:

> "Analyze my [Service Name] costs and suggest optimizations"

The guardian will apply the same analysis framework to any service.

## Safety

The guardian follows strict safety principles:

- **Never auto-executes** — All scripts are presented for your review
- **Destructive actions require confirmation** — Always asks before suggesting termination/deletion
- **Provides rollback commands** — Shows how to undo changes
- **Estimates cost savings** — Every recommendation includes expected savings

## License

MIT License — see [LICENSE](LICENSE)
