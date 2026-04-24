# Changelog

All notable changes to this project will be documented in this file.

## [1.1.0] - 2026-04-24

### Added
- Pod Security Admission (PSA) check (item 3.4) in access-identity assessment
- Pod Security Standards reference URL in report generation
- Concrete thresholds throughout steering files: OOMKilled event counts, CoreDNS scaling ratios, NodeLocal DNSCache cluster size guidance, HPA minReplicas context, maxUnavailable math for small deployments, CloudWatch log retention recommendation, LB deregistration delay alignment
- `logs:DescribeLogGroups` and `eks:DescribeAddonVersions` to README IAM permissions
- Python and environment patterns to .gitignore
- CHANGELOG.md

### Changed
- Version calendar in cluster-lifecycle.md now includes "Last verified" date and link to live AWS calendar
- CLAUDE.md expanded from 18 lines to full overview with prerequisites, workflow phases, and critical rules
- Cluster Autoscaler rated AMBER (legacy) vs Karpenter/Auto Mode rated GREEN (AWS-preferred path)
- Code block language tags now preserved in HTML converter output

### Removed
- Personal desktop file paths from .claude/settings.local.json

## [1.0.0] - 2026-04-02

### Added
- Initial release with 10 assessment areas and 37 check items
- Steering file architecture for modular check definitions
- Markdown to HTML report converter (zero dependencies)
- Pre-verified AWS documentation reference map
- Support for both AWS-managed and open-source EKS MCP servers
