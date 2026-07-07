# PAI Identity & Access Management (IAM) Architecture

## Unique ID Schema (UID)
Every entity in the PAI ecosystem MUST have a persistent Unique ID for audit logging, RBAC, and authorized task monitoring.

| Prefix | Category | Examples |
| :--- | :--- | :--- |
| **SYS-** | Systems & Nodes | `SYS-001` (NAS), `SYS-002` (M710q) |
| **AGT-** | AI Agents | `AGT-001` (PAI Nova Claude), `AGT-002` (PAI Nova Gemini), `AGT-003` (PAI Nova OpenCode) |
| **AST-** | Assets & Services | `AST-001` (Vaultwarden), `AST-002` (Authentik) |
| **USR-** | Human Users | `USR-001` (Duane) |

## Role-Based Access Control (RBAC)
Roles are defined in Authentik and mapped to Agent IDs.

1. **pai-admin** (Full access, secret rotation, system-level changes).
2. **pai-agent-privileged** (Access to specific production secrets, write access to memory).
3. **pai-agent-standard** (Read-only access to memory, no secret access).

## Secret Retrieval (Credential Guard)
Agents never access raw credential files. They must:
1. Present their **Agent ID**.
2. Call `fetch-secret.sh <secret_name>`.
3. The script verifies the Agent's group membership in Authentik/Vaultwarden before returning the secret to memory.
