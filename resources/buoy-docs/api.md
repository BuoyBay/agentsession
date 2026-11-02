---
title: "AgentSession API"
summary: "Session construction, polling, tool configuration, approvals, and explicit token handoff."
category: "Reference"
order: 20
keywords: ["ready", "busy", "cancel", "approval", "token", "mcp"]
symbols: ["AgentSession", "AgentSessionEvent", "AgentSession.Poll", "AgentSession.SendPrompt", "AgentSession.Close", "AgentSession.Cancel", "AgentSession.RespondApproval", "AgentSession.ConfigureMCPPromptTool", "AgentSession.ConfigureClaudeProjectServer", "AgentSession.CallTool", "AgentSession.AcceptChatGPTTokens", "AgentSession.RespondTokenRefresh"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

## Lifecycle

| Symbol | Contract |
| --- | --- |
| `AgentSession(endpoint, workingDirectory, workspace)` | Start the exact endpoint and initialize its protocol |
| Constructor overload with `environmentOverrides` and `resumeSessionId = ""` | Set variables for this child only; optionally resume an existing Codex thread |
| `Poll()` | Bounded nonblocking read/write progress; ordered `Array Of AgentSessionEvent` |
| `Ready`, `Busy`, `Failed`, `LastError`, `SessionId`, `IsClosed` | Read-only lifecycle snapshots |
| `SendPrompt(promptText, context)` | Start a turn; rejects before readiness or while busy; `Nil` context is allowed |
| `Cancel()` | Request protocol interruption; repeated calls are harmless |
| `ReadStderr()` / `StderrTruncated` | Drain raw diagnostics and detect the retained 1 MiB limit |
| `Close()` | Idempotently close/reap the direct child; also called by ARC destruction |
| `AgentSessionEvent.Kind`, `Text`, `RawJSON`, `RequestId` | Read-only event snapshots; raw JSON is a detached copy |

Common events include `ready`, `text_delta`, `thought_delta`, `tool_call`, `proposal`, `approval`, `turn_complete`, `turn_failed`, `turn_interrupted`, `token_refresh`, `error`, and `closed`. Provider extension fields stay in raw JSON. Outgoing authentication requests never become events.

## Tools and project permissions

| Method | Behavior |
| --- | --- |
| `RespondApproval(requestId, decision)` | Resolve a pending provider request or explicitly apply/decline an ACP project proposal |
| `ConfigureClaudeProjectServer(serverName)` | Enable SDK MCP project-tool handling for an explicitly registered CLI server name; call before polling |
| `CallTool(toolName, arguments)` | MCP-only call restricted to discovered tool names |
| `ConfigureMCPPromptTool(toolName, promptField, contextField = "")` | Map `SendPrompt` to one actual discovered MCP tool |
| Four-argument `ConfigureMCPPromptTool(..., fixedArguments)` | Retain a detached object such as an actual existing T3 thread ID; prompt/context overwrite only configured fields |

ACP file reads return entire registered buffers; line-range and limit requests reject. ACP writes emit `proposal` followed by `approval`; accept applies the proposal in memory, decline/cancel rejects it. For offered ACP permissions, return the actual option ID or an empty decision for cancelled. Native Codex/Claude escalation tools cannot be accepted through this adapter.

Claude SDK project tools require the CLI's real stream-json/control flags and matching explicit `--mcp-config` registration of a `type: "sdk"` server. The package adds neither registration nor a server name silently. MCP tools must be discovered; there is no fabricated native T3 chat protocol.

## Explicit authentication

`AcceptChatGPTTokens(accessToken, accountId, planType = "")` sends experimental Codex external-account tokens over private stdin. `RespondTokenRefresh(requestId, accessToken, accountId, planType = "")` answers a refresh event for the same route. They require an actual account identifier and the compatible agent schema; do not derive that ID from a Sign in with ChatGPT subject.

For the opaque-token/custom-provider flow, configure the provider explicitly and use per-child environment overrides. When its token rotates, close/restart the agent with the new environment and saved `SessionId`. The package does not discover credentials, mutate the global environment, or implement a refresh loop.

## Bounds and errors

JSONL framing defaults to 8 MiB lines and 128 JSON nesting levels. Pending stdin is bounded to 8 MiB/256 frames, outstanding RPCs to 128, approvals to 16/8 MiB, returned events to 1,024/8 MiB, and retained stderr to 1 MiB. Parsing, protocol, or transport failure closes the child and reports a failed session. The host remains responsible for process-tree and turn-timeout policy.

See the [getting-started guide](getting-started.md) and [overview](index.md).

[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
