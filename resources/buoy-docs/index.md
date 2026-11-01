---
title: "AgentSession"
summary: "Drive explicitly configured local coding-agent sessions against authoritative project buffers."
category: "Overview"
order: 0
keywords: ["agent", "process", "session", "project", "acp", "codex", "claude", "mcp"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# AgentSession

AgentSession combines AgentKit codecs, ProcessKit subprocess pipes, and ProjectKit's in-memory workspace into a live local agent session. It supports ACP, Codex app-server, Claude stream-json, and explicitly configured MCP tools. It never guesses an executable, command-line arguments, provider, remote URL, or T3 method.

## Where it fits

The host supplies an `AgentEndpoint`, working directory, and `ProjectWorkspace`. The constructor starts that exact child and begins initialization. Frequent `Poll()` calls advance nonblocking input/output and return ordered immutable `AgentSessionEvent` snapshots. The host owns UI, turn deadlines, process-tree policy, model selection, and permission decisions.

Registered workspace buffers are authoritative: project reads can see unsaved text. Project proposals remain revision-checked changes in memory. Session operations do not save project items to disk.

## Permissions and tools

Codex receives generated project list/read/propose tools from the actual `ProjectMCPHost` registry. ACP reads map only exact registered absolute paths; unsupported partial reads reject. ACP writes create a proposal and approval request instead of silently writing. The host explicitly accepts, declines, or cancels them.

The project adapter will not accept native Codex command/file escalation or native Claude tools; only denial/cancellation is allowed for those requests. Codex starts with read-only sandboxing and no automatic escalation. These bridge restrictions do not turn an external executable into an operating-system sandbox.

## Authentication boundary

Credentials are supplied explicitly, preferably through the constructor's per-child environment override. Session traffic is not logged by the package. Common credential fields are redacted from event JSON; raw stderr remains sensitive. A host must deliberately choose the authentication route supported by its agent version and account flow.

Read [Getting started](getting-started.md) for a handshake-only example and [API reference](api.md) for session state, project proposals, and MCP configuration.
