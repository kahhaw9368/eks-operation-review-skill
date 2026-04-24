# EKS Operation Review Skill

This is a Claude Code skill that performs automated EKS Operational Excellence assessments.

## How It Works

- `/eks-review` triggers the assessment workflow
- The skill uses two MCP servers defined in `.mcp.json`:
  - `awslabs.eks-mcp-server` — queries EKS cluster state (discovery, describe, Kubernetes API)
  - `awslabs.aws-documentation-mcp-server` — used during assessment for ad-hoc AWS doc lookups when steering file guidance is insufficient. **Not called during report generation** — the report uses a pre-verified reference map in `steering/report-generation.md` instead.
- Steering files in `steering/` contain per-section check instructions — read them before running each section
- `tools/report_to_html.py` converts the final markdown report to styled HTML

## Prerequisites

- AWS credentials configured with EKS read access (`aws sts get-caller-identity` should succeed)
- Python 3.10+ and uv installed
- See `README.md` for full IAM permission list and Kubernetes RBAC requirements

## Workflow Phases

1. **Pre-flight** — Verify MCP connectivity, list clusters, user selects one, describe and confirm
2. **Assessment (Steps 1-10)** — Load each steering file, execute checks, rate GREEN/AMBER/RED/UNKNOWN
3. **Report Generation (Step 11)** — Compile findings, validate consistency, write markdown report, optionally convert to HTML

## Critical Rules

- Load the steering file BEFORE executing checks for that section
- Only rate based on what was actually observed — never assume
- If a check fails or returns no data, mark UNKNOWN
- Every RED finding must have a specific, actionable recommendation
- Maintain consistency: if an item is RED in the findings table, it must be RED everywhere (executive summary, prioritized actions)
- Do NOT fabricate AWS documentation URLs — use the pre-verified reference map
