---
title: "agentsession typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["AgentSession", "AgentSessionEvent"]
dependencies: []
symbols: ["AgentSession", "AgentSessionEvent"]
related: ["api.md", "getting-started.md", "index.md"]
---

# agentsession typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## AgentSessionEvent

`Public Class AgentSessionEvent`

Event snapshots contain no outgoing token request payloads.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Kind` | `String` | Yes | No | — | — |
| `Text` | `String` | Yes | No | — | — |
| `RawJSON` | `JSONItem` | Yes | No | — | — |
| `RequestId` | `JSONItem` | Yes | No | — | — |

## AgentSession

`Public Class AgentSession`

Explicit local subprocess session; call Poll frequently while draining stderr separately.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Ready` | `Boolean` | Yes | No | — | — |
| `Busy` | `Boolean` | Yes | No | — | — |
| `Failed` | `Boolean` | Yes | No | — | — |
| `LastError` | `String` | Yes | No | — | — |
| `SessionId` | `String` | Yes | No | — | — |
| `IsClosed` | `Boolean` | Yes | No | — | — |
| `StderrTruncated` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(endpoint As AgentEndpoint, workingDirectory As String, workspace As ProjectWorkspace)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `endpoint` | `AgentEndpoint` | `ByVal` | Required |
| `workingDirectory` | `String` | `ByVal` | Required |
| `workspace` | `ProjectWorkspace` | `ByVal` | Required |

Returns: no value.

### Constructor (overload 2)

`Public Sub Constructor(endpoint As AgentEndpoint, workingDirectory As String, workspace As ProjectWorkspace, environmentOverrides As Map Of String, String, resumeSessionId As String = "")`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `endpoint` | `AgentEndpoint` | `ByVal` | Required |
| `workingDirectory` | `String` | `ByVal` | Required |
| `workspace` | `ProjectWorkspace` | `ByVal` | Required |
| `environmentOverrides` | `Map Of String, String` | `ByVal` | Required |
| `resumeSessionId` | `String` | `ByVal` | `""` |

Returns: no value.

### ConfigureClaudeProjectServer

`Public Sub ConfigureClaudeProjectServer(serverName As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `serverName` | `String` | `ByVal` | Required |

Returns: no value.

### Poll

`Public Function Poll() As Array Of AgentSessionEvent`

Drain both process streams and retry bounded pending stdin; never interpret stderr as protocol.

No parameters.

#### Returns

| Type |
| --- |
| `Array Of AgentSessionEvent` |

### ReadStderr

`Public Function ReadStderr() As Bytes`

No parameters.

#### Returns

| Type |
| --- |
| `Bytes` |

### SendPrompt

`Public Sub SendPrompt(promptText As String, context As JSONItem)`

Context is explicit JSON serialized into a separate text contentBlock; it never changes host permissions.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `promptText` | `String` | `ByVal` | Required |
| `context` | `JSONItem` | `ByVal` | Required |

Returns: no value.

### ConfigureMCPPromptTool

`Public Sub ConfigureMCPPromptTool(toolName As String, promptField As String, contextField As String = "")`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `toolName` | `String` | `ByVal` | Required |
| `promptField` | `String` | `ByVal` | Required |
| `contextField` | `String` | `ByVal` | `""` |

Returns: no value.

### ConfigureMCPPromptTool (overload 2)

`Public Sub ConfigureMCPPromptTool(toolName As String, promptField As String, contextField As String, fixedArguments As JSONItem)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `toolName` | `String` | `ByVal` | Required |
| `promptField` | `String` | `ByVal` | Required |
| `contextField` | `String` | `ByVal` | Required |
| `fixedArguments` | `JSONItem` | `ByVal` | Required |

Returns: no value.

### CallTool

`Public Sub CallTool(toolName As String, arguments As JSONItem)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `toolName` | `String` | `ByVal` | Required |
| `arguments` | `JSONItem` | `ByVal` | Required |

Returns: no value.

### Cancel

`Public Sub Cancel()`

No parameters.

Returns: no value.

### RespondApproval

`Public Sub RespondApproval(requestId As JSONItem, decision As String)`

Approval never escalates Codex native commands/files; project proposals are applied only here on explicit accept.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |
| `decision` | `String` | `ByVal` | Required |

Returns: no value.

### AcceptChatGPTTokens

`Public Sub AcceptChatGPTTokens(accessToken As String, accountId As String, planType As String = "")`

Experimental external-token handoff over private stdin; no argv/env/log persistence or automatic refresh.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `accessToken` | `String` | `ByVal` | Required |
| `accountId` | `String` | `ByVal` | Required |
| `planType` | `String` | `ByVal` | `""` |

Returns: no value.

### RespondTokenRefresh

`Public Sub RespondTokenRefresh(requestId As JSONItem, accessToken As String, accountId As String, planType As String = "")`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |
| `accessToken` | `String` | `ByVal` | Required |
| `accountId` | `String` | `ByVal` | Required |
| `planType` | `String` | `ByVal` | `""` |

Returns: no value.

### Close

`Public Sub Close()`

No parameters.

Returns: no value.

### Destructor

`Public Sub Destructor()`

No parameters.

Returns: no value.
