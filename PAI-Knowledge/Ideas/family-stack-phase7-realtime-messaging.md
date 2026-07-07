---
title: "Family Stack Phase 7 — Real-Time Agent Messaging"
type: idea
tags: [family-stack, real-time-messaging, redis, inter-agent, phase-7]
created: 2026-05-25
updated: 2026-05-25
quality: 5
source_session: 20260525-000000_phase7-realtime-messaging
related:
  - slug: family-stack-phase1-6
    type: extends
  - slug: pai-architecture
    type: part-of
---

# Family Stack Phase 7 — Real-Time Agent Messaging

## Thesis
PAI engines (PNC, PNK, PNX) communicate in real-time via Redis pub/sub + list queue, replacing async mailbox-only coordination.

## Architecture
- **Redis:** prism-redis container at 172.20.0.3:6379
- **Channels:** agent:pnc, agent:pnk, agent:pnx, agent:broadcast
- **Queue:** LPUSH/RPOP list pattern (pub/sub doesn't flush through docker exec)
- **JSON escaping:** Fixed via temp file pipe (`redis-cli -x LPUSH < file`)
- **Security:** HMAC-SHA256 signed messages with shared secret
- **Polling:** 500ms interval, 30s heartbeat

## Daemons
- **PNK:** PID 3248364, inbox at ~/.claude/.../agent-inbox/pnk/
- **PNC:** PID 3248365, inbox at ~/.claude/.../agent-inbox/pnc/
- **PNX:** PID 3248366, inbox at ~/.codex/memories/agent-inbox/pnx/

## Tools
- **AgentMessenger.ts:** CLI for send/listen/status/history
- **AgentMessengerDaemon.ts:** Long-running daemon with per-agent inbox paths
- **InboxCheck.hook.ts:** PNX inbox hook (~/.codex/hooks/) for SessionStart + UserPromptSubmit

## Integration
- StackMonitor.ts broadcasts alerts via Redis
- Phase completion notifications use real-time channel
- PNG deferred (being upgraded)

## Key Decisions
- List-based queue over pub/sub (docker exec flushing issue)
- Temp file pipe for JSON escaping (shell corruption)
- Per-agent inbox paths (PNX uses ~/.codex/ base)
- Mailbox fallback when Redis unreachable
