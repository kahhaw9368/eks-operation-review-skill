# Access & Identity

## Purpose
Assess IAM and RBAC configuration for security and operational excellence — pod-level permissions, least privilege, and API server access controls.

## Checks to Execute

### 3.1 — Pod-Level AWS Permissions (IRSA / EKS Pod Identity)

**What to check:**
- OIDC provider configured (prerequisite for IRSA)
- EKS Pod Identity associations
- Service accounts with IRSA annotations (`eks.amazonaws.com/role-arn`)
- Node IAM role policies (are they overly broad?)
- AWS credentials in Kubernetes Secrets or pod env vars

**How to check:**
1. Describe cluster → `identity.oidc.issuer` (OIDC configured?)
2. List Pod Identity associations
3. List ServiceAccounts across all namespaces → filter for IRSA annotation
4. List node groups → describe first one → get `nodeRole` → get policies for that role
5. List Secrets across namespaces → check for `AWS_ACCESS_KEY_ID` keys (⚠️ avoid reading Secret values)
6. List pods → check container env vars for `AWS_ACCESS_KEY_ID`

**Rating:**
- 🟢 GREEN: All AWS-accessing pods use IRSA or Pod Identity, node role is minimal, no hardcoded credentials
- 🟡 AMBER: IRSA partially adopted, or node role has some extra permissions
- 🔴 RED: No IRSA/Pod Identity, node role has broad permissions (S3FullAccess, DynamoDBFullAccess), or hardcoded AWS credentials found
- ⬜ UNKNOWN: Cannot determine which pods need AWS access vs which don't

**Key talking point:** Node-level IAM = every pod on that node inherits the same permissions. One compromised pod gets access to everything.

---

### 3.2 — Least Privilege RBAC

**What to check:**
- ClusterRoleBindings to cluster-admin (count and subjects)
- ClusterRoles with wildcard permissions (`*` verbs on `*` resources)
- Ratio of namespace-scoped RoleBindings vs cluster-scoped ClusterRoleBindings
- Service accounts with cluster-admin

**How to check:**
1. List ClusterRoleBindings → filter `roleRef.name == "cluster-admin"` → count and list subjects
2. List ClusterRoles → check rules for `verbs: ["*"]` and `resources: ["*"]`
3. Count RoleBindings across all namespaces vs ClusterRoleBindings
4. List application namespaces (exclude kube-system, kube-public, kube-node-lease, default)

**Rating:**
- 🟢 GREEN: Namespace-scoped RBAC, cluster-admin limited to 1-2 break-glass bindings, periodic review
- 🟡 AMBER: Some namespace isolation but cluster-admin overused (>3 bindings)
- 🔴 RED: Developers have cluster-admin in production, wildcard service accounts, no namespace isolation
- ⬜ UNKNOWN: Cannot determine if RBAC is reviewed periodically — suggest user investigate

---

### 3.3 — EKS API Server Endpoint & Network Access

**What to check:**
- Public/private endpoint configuration
- Public access CIDR restrictions
- Cluster security group inbound rules
- Whether audit logging is enabled

**How to check:**
1. Describe cluster → `resourcesVpcConfig.endpointPublicAccess`, `endpointPrivateAccess`, `publicAccessCidrs`
2. Describe cluster → `logging.clusterLogging` (check if audit log type is enabled)

**Rating:**
- 🟢 GREEN: Private endpoint enabled, public either disabled or CIDR-restricted, audit logging on
- 🟡 AMBER: Public endpoint with CIDR restrictions, or private enabled but audit logging off
- 🔴 RED: Public endpoint open to `0.0.0.0/0`, or no audit logging
- ⬜ UNKNOWN: Cannot determine MFA/SSO requirements — suggest user investigate

**Key talking point:** An API server open to 0.0.0.0/0 is exposed to the internet. You're relying entirely on authentication.
