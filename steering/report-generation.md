# Report Generation

## Purpose
After all section checks are complete, generate the EKS Operation Review report.

## Consistency Checks (MANDATORY before writing)

Before writing the report, validate consistency:

1. **Build a master list** of all findings with their ratings from sections 01-10
2. **For each RED item:** confirm it appears in "Critical" or "Important" prioritized actions
3. **For each AMBER item:** confirm it appears in "Important" or "Quick Wins"
4. **For the Executive Summary:** only mention ratings that match the master list — do not call something a "critical gap" if it's AMBER, or omit a RED from the summary
5. **For Prioritized Actions:** every entry must reference the finding ID (e.g., "4.1 — Control Plane Logging")

## Workflow

### Step 1: Build Master Finding List

```
| Section | Item ID | Item Name | Rating |
```

### Step 2: Calculate Maturity Score

- Count GREEN, AMBER, RED, UNKNOWN
- Calculate percentages (exclude UNKNOWN from denominator)

### Step 3: Write Executive Summary

From the master list, identify:
- **Top strengths** (GREEN items with highest operational impact)
- **Top gaps** (RED items, ordered by blast radius: security > availability > cost)
- Write 2-3 paragraphs. Every rating mentioned must match the master list.

### Step 4: Write Findings Tables

One table per section. Every item from the master list must appear.

### Step 5: Write Prioritized Actions

Cross-reference against the master list:
- **Critical (30 days):** All RED items. Column: `Finding | Action | References`
- **Important (90 days):** All AMBER items. Column: `Finding | Action | References`
- **Quick Wins:** Items (RED or AMBER) fixable in < 1 hour. Column: `Finding | Action | Effort | Impact | References`

Every entry must include the finding ID and name (e.g., "4.1 — Control Plane Logging 🔴").

### Step 6: Write Investigate Manually

All UNKNOWN items with specific questions the user should answer.

### Step 7: Look Up AWS References

Try using the AWS Documentation MCP server to find verified URLs for each recommendation.

If the AWS Documentation MCP server is not available or fails, fall back to these well-known URLs — do NOT skip references entirely:
- EKS Best Practices Guide: `https://docs.aws.amazon.com/eks/latest/best-practices/`
- EKS User Guide: `https://docs.aws.amazon.com/eks/latest/userguide/`
- EKS Version Calendar: `https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html`
- AWS Containers Blog: `https://aws.amazon.com/blogs/containers/`

Do NOT fabricate deep-link URLs. When in doubt, link to the broad section page.

### Step 8: Final Consistency Validation

Before outputting, scan the report for:
- Any RED item missing from Prioritized Actions → add it
- Any item mentioned in Executive Summary with wrong rating → fix it
- Any Prioritized Action without a finding ID → add the ID

### Step 9: Write the Report File

Write the report to the **workspace directory**. The file must be created inside the current workspace.

**Filename format:** `EKS-Operation-Review-<cluster-name>-<YYYY-MM-DD>-<HHMM>.md`

**Example:** `EKS-Operation-Review-demo-cluster-2026-03-22-1830.md`

The file should be written to the workspace root or a `reports/` subfolder within the workspace. Do NOT use absolute paths outside the workspace.

### Step 10: Offer HTML Conversion

Ask: "Would you like me to convert the report to HTML?"

If yes, run the conversion script — do NOT generate HTML manually. Execute this command:

```bash
python3 report_to_html.py <report-filename>.md
```

Run this from the workspace root where `report_to_html.py` is located. If the script is not found in the workspace root, check `tools/report_to_html.py`.

Do NOT create HTML by hand. Always use the script.
