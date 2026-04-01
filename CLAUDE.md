# EKS Operation Review Skill

This is a Claude Code skill that performs automated EKS Operational Excellence assessments.

## How It Works

- `/eks-review` triggers the assessment workflow
- The skill uses two MCP servers defined in `.mcp.json`:
  - `awslabs.eks-mcp-server` — queries EKS cluster state
  - `awslabs.aws-documentation-mcp-server` — looks up verified AWS doc URLs
- Steering files in `steering/` contain per-section check instructions — read them before running each section
- `tools/report_to_html.py` converts the final markdown report to styled HTML

## Prerequisites

- AWS credentials configured with EKS read access
- Python 3.10+ and uv installed
