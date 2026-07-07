# PAI Org Session Management

The `org-session-manager.py` utility is used to manage org-scoped RBAC session YAML state without requiring manual file edits. It resides under `~/.claude/PAI/Tools/org-session-manager.py`.

Sessions are stored in subdirectory states under `~/.claude/PAI/MEMORY/STATE/org-sessions/`:
- `active/` — Active mutable sessions.
- `closed/` — Successfully closed sessions (immutable).
- `revoked/` — Revoked/terminated sessions (immutable).
- `examples/` — Static read-only reference templates.

---

## Command Reference

### 1. Create a Session
Create a new active session under the `active/` directory.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py create \
  --structure business \
  --team-name development \
  --created-by AGT-001
```
*Creates: `ORG-<timestamp>-development.yml`*

### 2. Assign a Role
Assign a role from the org RBAC contract to an actor in an active session.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py assign \
  ORG-20260618-014929-development \
  --uid AGT-003 \
  --role Engineer \
  --ttl-minutes 120
```

### 3. Suspend an Assignment
Suspend a role assignment inside an active session. The validator will deny mutating operations for suspended assignments.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py suspend \
  ORG-20260618-014929-development \
  --assignment-id ORA-20260618-014937-001 \
  --reason "Pending security review"
```

### 4. Revoke an Assignment
Permanently revoke a role assignment in an active session.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py revoke-assignment \
  ORG-20260618-014929-development \
  --assignment-id ORA-20260618-014937-001 \
  --reason "Project scope completed"
```

### 5. Close a Session
Close the session and move its file from `active/` to the `closed/` directory.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py close \
  ORG-20260618-014929-development
```

### 6. Revoke a Session
Revoke the entire session and move its file to `revoked/` with an audit reason.
```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py revoke-session \
  ORG-20260618-014929-development \
  --reason "Compromised credential event"
```

### 7. Show Session State
Show the YAML or JSON contents of any session.
```bash
# YAML (default)
python3 ~/.claude/PAI/Tools/org-session-manager.py show ORG-20260618-014929-development

# JSON format
python3 ~/.claude/PAI/Tools/org-session-manager.py show ORG-20260618-014929-development --json
```

---

## Authorization & Audit Preview

You can test authorization rules against any session using the `audit-preview` command. It runs a dry-run auth decision via `validate-org-rbac.py`.

```bash
python3 ~/.claude/PAI/Tools/org-session-manager.py audit-preview \
  ORG-20260618-014929-development \
  --actor-uid AGT-003 \
  --action task:update:owned \
  --resource TASK-001
```

This resolves the session automatically across lifecycle folders and outputs the dry-run decision trace.
