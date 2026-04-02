# EKS Operation Review Skill

A Claude Code skill that performs automated EKS operational excellence assessments. It connects to your live EKS cluster, checks 37 items across 10 operational areas — informed by the [EKS Best Practices Guide](https://docs.aws.amazon.com/eks/latest/best-practices/) and [EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/) — and produces a rated report with prioritized recommendations.

**Read-only** -- this skill does not modify your cluster. All operations are describe/list/get calls only.

## Sample Output

The skill generates an HTML report with an executive summary, maturity score, detailed findings, and prioritized actions. A typical assessment takes **5-10 minutes** per cluster.

![Executive Summary and Maturity Score](docs/sample-report-summary.png)

![Detailed Findings by Section](docs/sample-report-findings.png)

## What Problem Does It Solve

Reviewing an EKS cluster for operational readiness is time-consuming and easy to miss things. This skill automates ~70-75% of those checks — drawing on guidance from the [EKS Best Practices Guide](https://docs.aws.amazon.com/eks/latest/best-practices/) and [EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/) — by querying your cluster directly, rating each item as GREEN/AMBER/RED, and generating a report with specific, actionable recommendations linked to AWS documentation.

## What Gets Assessed

| # | Area | Examples |
|---|------|----------|
| 01 | Cluster Lifecycle | Version currency, upgrade readiness, deprecated APIs |
| 02 | Infrastructure as Code | IaC provenance, GitOps tools, drift detection |
| 03 | Access & Identity | IRSA/Pod Identity, RBAC, API server endpoint |
| 04 | Observability | Control plane logging, metrics, log aggregation, alerting |
| 05 | Workload Configuration | Resource requests, health probes, PDBs, image tags |
| 06 | Networking | IP capacity, CoreDNS, network policies |
| 07 | Autoscaling | Karpenter/CA, HPA, topology spread |
| 08 | Deployment Practices | Rollout strategy, CI/CD, graceful shutdown |
| 09 | Operational Processes | Backup/DR, runbooks, on-call |
| 10 | Add-on Management | Managed add-ons, node health monitoring, cluster insights |

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- Python 3.10+ and [uv](https://docs.astral.sh/uv/getting-started/installation/) installed
- AWS credentials configured (`aws sts get-caller-identity` should work)

## MCP Server Setup

This skill requires two MCP servers. The configuration lives in `.mcp.json` at the project root.

### EKS MCP Server

The EKS MCP server provides tools for querying cluster state, listing resources, and inspecting configurations.

**Recommended: AWS-Managed EKS MCP Server (Preview)**

The [AWS-managed EKS MCP server](https://docs.aws.amazon.com/eks/latest/userguide/eks-mcp-introduction.html) is a hosted service with automatic updates, CloudTrail audit logging, and a built-in troubleshooting knowledge base. It provides better quality results than the open-source version.

1. Attach the `AmazonEKSMCPReadOnlyAccess` managed policy to your IAM user/role
2. Update `.mcp.json` (replace `{region}` with your AWS region):

```json
{
  "mcpServers": {
    "eks-mcp": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://eks-mcp.{region}.api.aws/mcp",
        "--service", "eks-mcp",
        "--profile", "default",
        "--region", "{region}",
        "--read-only"
      ]
    }
  }
}
```

See the [Getting Started guide](https://docs.aws.amazon.com/eks/latest/userguide/eks-mcp-getting-started.html) for full setup instructions.

**Alternative: Open-Source EKS MCP Server**

The project ships with the open-source [awslabs.eks-mcp-server](https://github.com/awslabs/mcp) as the default in `.mcp.json`. This works out of the box with no additional IAM setup, but lacks the managed server's built-in troubleshooting knowledge base and auto-updates.

```json
{
  "mcpServers": {
    "awslabs.eks-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.eks-mcp-server@latest"],
      "env": { "FASTMCP_LOG_LEVEL": "ERROR" }
    }
  }
}
```

### AWS Documentation MCP Server

Used to look up verified AWS documentation URLs for report recommendations. Already configured in `.mcp.json`:

```json
{
  "mcpServers": {
    "awslabs.aws-documentation-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": { "FASTMCP_LOG_LEVEL": "ERROR" }
    }
  }
}
```

## Required Permissions

### AWS IAM

The skill is **read-only**. Minimum IAM permissions needed:

```
eks:ListClusters, eks:DescribeCluster, eks:ListNodegroups,
eks:DescribeNodegroup, eks:ListAddons, eks:DescribeAddon,
eks:ListInsights, eks:DescribeInsight, eks:ListAccessEntries,
eks:ListPodIdentityAssociations
ec2:DescribeSubnets, ec2:DescribeVpcs
iam:ListAttachedRolePolicies, iam:ListRolePolicies,
iam:GetPolicy, iam:GetPolicyVersion
cloudwatch:DescribeAlarms
```

If using the AWS-managed EKS MCP server, attach the `AmazonEKSMCPReadOnlyAccess` managed policy instead.

### Kubernetes RBAC

Your IAM identity also needs read access to Kubernetes resources (Nodes, Pods, Deployments, Services, etc.) via an EKS access entry or `aws-auth` ConfigMap.

## Getting Started

```bash
git clone https://github.com/kahhaw9368/eks-operation-review-skill.git
cd eks-operation-review-skill
```

Then open Claude Code from this directory:

```bash
claude
```

Once inside Claude Code, run:

```
/eks-review
```

The skill will discover your EKS clusters, ask you to pick one, and walk you through the assessment.

## Output

Reports are generated in the workspace root:
- `EKS-Operation-Review-<cluster>-<date>.md` -- full Markdown report
- `EKS-Operation-Review-<cluster>-<date>.html` -- styled HTML (optional)

## Limitations

- **One cluster at a time** -- the skill assesses a single cluster per run. Run it again for additional clusters.
- **Process questions are UNKNOWN** -- items like runbooks, on-call rotation, and post-incident reviews (Section 09) cannot be detected from cluster state. These are marked UNKNOWN with questions for you to investigate.
- **Point-in-time snapshot** -- the assessment reflects cluster state at the time of the run. It does not monitor ongoing changes.
- **Requires cluster access** -- your IAM identity must have both AWS API permissions and Kubernetes RBAC access to the target cluster.

## Troubleshooting

**MCP server not responding**

If the skill fails to list clusters, try:

1. Check Python and uv are installed: `uv --version`
2. Check AWS credentials work: `aws sts get-caller-identity`
3. Test the MCP server directly: `uvx awslabs.eks-mcp-server@latest`
4. Verify `AWS_PROFILE` and `AWS_REGION` in `.mcp.json` match your environment

**No clusters found**

The skill lists clusters in the region configured in your AWS credentials. To target a different region, set `AWS_REGION` in `.mcp.json` or your environment.

**Permission denied errors**

Ensure your IAM identity has the permissions listed in [Required Permissions](#required-permissions) and has a Kubernetes RBAC binding (via EKS access entry or `aws-auth` ConfigMap).

## Project Structure

```
.claude/commands/eks-review.md   # Skill entry point
CLAUDE.md                        # Instructions for Claude Code
.mcp.json                        # MCP server configuration
steering/                        # Per-section check instructions
  cluster-lifecycle.md
  infrastructure-as-code.md
  access-identity.md
  observability.md
  workload-configuration.md
  networking.md
  autoscaling.md
  deployment-practices.md
  operational-processes.md
  addon-management.md
  report-generation.md
tools/report_to_html.py          # Markdown to HTML converter
reports/                         # Alternative report output directory
```

## Customization

To use a specific AWS profile or region, edit `.mcp.json`:

```json
"env": {
  "AWS_PROFILE": "your-profile",
  "AWS_REGION": "us-west-2"
}
```

## Contributing

Contributions are welcome. Please open an issue first to discuss what you'd like to change.

## License

MIT
