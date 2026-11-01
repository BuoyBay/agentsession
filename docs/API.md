# AgentSession · 0.1.0

Live local agent sessions for Buoy Studio, built on `agentkit` JSONL codecs,
`processkit` subprocess pipes and `projectkit` in-memory items. The package drives
ACP, Codex app-server, Claude stream-json, and explicitly configured MCP tools.
It never guesses an executable, CLI arguments, provider, T3 method, or remote URL.

## Build and verify


Copy the `agentkit`, `processkit`, `editorkit`, `projectkit` and `agentsession`
archives into a consumer's `packages/`. Use `Import agentsession`, `Import agentkit`
and `Import projectkit`. The [example](../examples/agentsession_demo.bui)
performs an explicit ACP handshake without sending a model prompt.

The fixture runner launches only the checked-in Python protocol peer. It does
not invoke Claude, Codex, T3, a cloud model or a real authentication endpoint.

## Session API

| API | Behavior |
| --- | --- |
| `AgentSession(endpoint, workingDirectory, workspace)` | Launch the exact `AgentEndpoint` executable/argv and begin initialization. |
| `AgentSession(endpoint, workingDirectory, workspace, childEnvironment, resumeSessionId = "")` | Override named environment variables for this child only; optionally resume an existing Codex thread. |
| `Poll()` | Drain stdout/stderr, retry partial stdin writes, and return ordered `AgentSessionEvent` snapshots. |
| `Ready`, `Busy`, `Failed`, `LastError`, `SessionId`, `IsClosed` | Inspect lifecycle state. |
| `SendPrompt(promptText, context)` | Start a turn; `Nil` omits explicit JSON context. |
| `Cancel()` | Request protocol cancellation; repeated cancellation is harmless. |
| `RespondApproval(requestId, decision)` | Answer a pending provider permission or explicitly apply/decline an ACP project proposal. |
| `ReadStderr()` | Drain retained raw stderr bytes independently of protocol events. |
| `StderrTruncated` | Whether diagnostics exceeded the retained 1 MiB limit. |
| `Close()` | Idempotently close/reap the direct child. ARC also closes it. |

Call `Poll` frequently from a timer or worker, including while waiting for
approval or retrying cancellation. `Ready` means initialization succeeded;
`Busy` separately indicates an in-flight turn/tool. A second prompt is rejected
while busy. Cancellation waits for the provider's completion acknowledgement;
closing the session forcibly ends the direct child if necessary. Startup has a
30-second deadline; the application owns turn deadlines and process-tree policy.

Events expose read-only `Kind`, `Text`, `RawJSON` and `RequestId` snapshots.
Common kinds are `ready`, `text_delta`, `thought_delta`, `tool_call`, `proposal`,
`approval`, `turn_complete`, `turn_failed`, `turn_interrupted`, `token_refresh`,
`error` and `closed`. Provider-specific fields remain available in `RawJSON`.
Outgoing authentication requests never become events. Common credential field
names are redacted from event JSON. The session does not print protocol traffic
or diagnostics; callers should treat raw provider stderr as sensitive data.

## Authoritative project buffers

Codex advertises three generated dynamic tools: `project_list_items`,
`project_read_item`, and `project_propose_edits`. They call `ProjectMCPHost`,
so reads reflect registered unsaved buffers rather than stale disk contents.
Tool definitions are obtained from the actual host registry. Proposed changes
produce `proposal` events with `ProposalJSON` data, including the proposal ID,
old/new text and expected versions. They remain pending until the application
explicitly calls `ProjectWorkspace.ApproveProposal` or rejects them.

ACP `fs/read_text_file` maps only exact registered absolute paths under the
chosen working directory. It returns the complete current buffer; optional
partial line/limit requests are explicitly rejected. `fs/write_text_file`
creates a full-buffer replacement proposal and emits `proposal` then `approval`.
The write request remains pending until `RespondApproval(id, "accept")` applies
the proposal to the in-memory workspace and replies, or `"decline"`/`"cancel"`
rejects it. No session operation saves a project item to disk.

ACP permission decisions must be an offered option ID; an empty decision means
cancelled. Native Codex command/file escalation and native Claude tools cannot
be accepted through this project adapter: only `decline` or `cancel` is allowed.
Unknown host requests receive protocol errors. Codex threads request read-only
sandboxing and no automatic escalation. The selected native agent executable
still owns its own runtime capabilities; the project bridge is not an OS sandbox.

## Claude SDK project tools

Use an explicit Claude stream-json endpoint, including the CLI's documented
stream input/output/control flags. To expose project tools, register an SDK MCP
server in the endpoint's explicit `--mcp-config` JSON with its chosen name and
`type: "sdk"`, then call `ConfigureClaudeProjectServer(theSameName)` before polling.
The adapter handles `mcp_message` control requests only for that registered name.
Project read/list/proposal permissions can be satisfied through this bridge;
other native tools require denial. CLI registration is never silently added.

The control shapes follow the official [Claude Agent SDK implementation](https://github.com/anthropics/claude-agent-sdk-python/blob/main/src/claude_agent_sdk/_internal/query.py).
Initializing stream-json, prompting, streamed output, SDK MCP, tool denial and
interrupt requests are fixture-tested. Endpoint CLI flags/version compatibility
remain the caller's explicit configuration.

## T3 and other MCP servers

MCP sessions perform initialization and tool discovery; the `tools` event carries
the actual registry. Generic `CallTool(toolName, arguments)` accepts only discovered
tools. There is no assumed T3 chat protocol or automatically launched thread.

`ConfigureMCPPromptTool(toolName, promptField, contextField = "")` enables
`SendPrompt` for a specific discovered tool and declared argument fields.
The overload `ConfigureMCPPromptTool(toolName, promptField, contextField,
fixedArguments)` defensively copies an explicit object such as a known T3
`threadId`. Sending a prompt begins with those fixed arguments and overwrites
only the configured prompt/context fields. This lets a host target an existing
T3 thread using its actual tool contract; the adapter never invents a thread ID
or private tool name. MCP cancellation uses the standard cancellation notification.

## Codex authentication and resuming

Codex fields were checked against installed `codex-cli 0.165.0` using
`codex app-server generate-json-schema --experimental`, plus the official
[app-server documentation](https://developers.openai.com/codex/app-server/).
Initialization opts into experimental APIs; dynamic tool specifications include
`type: "function"`, and responses contain `contentItems` with `inputText` blocks
and a `success` flag. Threads are durable so a restarted process can resume them.
Resume retains previously registered tools rather than sending a field absent
from the installed `thread/resume` schema.

For a host-owned ChatGPT login that supplies an actual account identifier,
`AcceptChatGPTTokens(accessToken, accountId, planType = "")` calls experimental
`account/login/start` with `chatgptAuthTokens` over private stdin. A `token_refresh`
event can be answered with `RespondTokenRefresh(id, token, accountId, planType)`.
There is no automatic token discovery, account-ID derivation or refresh loop.

The newer [Sign in with ChatGPT app-server flow](https://developers.openai.com/siwc/token-sharing-open-source/codex-app-server)
uses an opaque access token and custom Responses-provider configuration. Do not
reinterpret its subject as a ChatGPT account ID. Use the child-environment
constructor to deliver the token to the explicitly configured provider's env key,
then close/restart with a renewed child environment and the saved `SessionId`.
Tokens are never placed in CLI argv, parent/global environment or application logs
by this package. The installed external-account-token schema marks that separate
API unstable; hosts must deliberately choose the matching authentication route.

## Bounds and failures

The JSONL codec bounds lines to 8 MiB and rejects malformed UTF-8, raw controls,
unsupported JSON NUL escapes and nesting over 128 before native parsing. Pending
stdin is bounded to 8 MiB/256 frames, outstanding RPCs to 128, pending approvals to
16/8 MiB, and returned events to 1,024/8 MiB. Each poll performs bounded nonblocking
reads/writes and drains stderr separately; failed parsing/protocol/transport closes
the direct child and reports a failed session. No process output is fed to a shell.

An `error` event alone does not mean the session is unusable. A rejected provider
request can emit `error` while `Failed` remains false; check `Ready` and `Busy`
before offering another prompt. Fatal protocol, transport, or initialization
errors detected by `Poll()` set `Failed` and `IsClosed`, clear `Ready` and `Busy`,
retain `LastError`, and emit an `error` event. Further polling does not restart the child. Construct
a new session when the host chooses to retry; prompts are not replayed.

For Codex resume, keep the saved thread ID separately until the new session is
successfully ready. A rejected resume can leave `SessionId` holding the peer's
unvalidated response, so never replace the saved ID from a failed session.
Resume again with the original saved ID after correcting the cause. Resuming a
thread does not establish that an interrupted turn completed; do not blindly
resend its prompt, which could duplicate work.

The fixture tests cover all four handshakes, Unicode fragmentation, backpressure
through a large prompt, cancellation, separate stderr, diagnostic bounds,
authoritative unsaved reads, explicit proposal approval without disk writes,
unmapped tool denial, request correlation, fixed MCP arguments, synthetic token
handoff/refresh and isolated child environment/resume. Module/test/example sources
are compiler-checked on six Buoy targets. Actual execution uses local fixture
processes on the platforms recorded by the combined Studio verification report.

Test suites and fixtures are maintainer-local and are not part of this public repository.
