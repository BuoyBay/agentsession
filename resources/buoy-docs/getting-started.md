---
title: "AgentSession getting started"
summary: "Launch a selected ACP executable, poll its handshake, and close the direct child."
category: "Guides"
order: 10
keywords: ["handshake", "poll", "close", "workspace", "endpoint"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Install the `agentkit`, `processkit`, `editorkit`, `projectkit`, and `agentsession` archives in the consumer's `packages/`. Import the specific modules used by your host. Native ProcessKit runtime slices must match the target platform.

## Perform a bounded ACP handshake

This complete example takes an explicit executable and absolute project folder. Any further arguments belong to that executable. It registers an unsaved scratch buffer, polls initialization for at most 30 seconds, and closes the session without sending a model prompt.

```basic
Import agentsession
Import agentkit
Import projectkit

' Explicitly launch an ACP endpoint. This example does not submit a prompt.
Sub Main(args As Array Of String)
    If Array.Count(args) < 3 Then
        Print "Usage: agentsession-demo /absolute/agent /absolute/project [agent arguments...]"
        Exit Sub
    End If
    Dim arguments As Array Of String = []
    For index As Integer = 3 To Array.Count(args) - 1
        Array.Append(arguments, args(index))
    Next
    Dim workspace As New ProjectWorkspace()
    workspace.RegisterItem("scratch", "scratch.bui", "source", "' Unsaved project buffer\n")
    Dim endpoint As New AgentEndpoint("Explicit ACP agent", AgentProtocol.ACP, args(1), arguments)
    Dim session As New AgentSession(endpoint, args(2), workspace)
    Dim deadline As Integer = System.Microseconds() + 30000000
    While Not session.Ready And Not session.Failed
        For Each update As AgentSessionEvent In session.Poll()
            Print update.Kind + ": " + update.Text
        Next
        If System.Microseconds() > deadline Then Exit While
        Thread.Sleep(5)
    End While
    If session.Failed Then Print session.LastError
    session.Close()
End Sub
```

Supply an actual ACP-compatible executable and the flags it requires. The package does not detect CLI dialects or install agents. The example's positional arguments follow Buoy's `args(0)` executable-name convention.

## Keep advancing the session

In an IDE, use a timer or worker rather than a blocking UI loop. Continue polling during prompts, pending approval, partial writes, and cancellation. `Ready` means initialization succeeded; `Busy` independently means a turn or tool request is in flight. A second prompt while busy rejects.

Call `SendPrompt(promptText, context)` only after readiness; `Nil` omits explicit JSON context. Present `approval` and `proposal` events to the user and retain their request/proposal identifiers. For ACP file proposals, `RespondApproval(id, "accept")` applies the in-memory revision-checked proposal and replies to the provider. An ACP permission decision must instead be one of that request's offered option IDs.

## Close and diagnose

`Cancel()` requests protocol cancellation and waits for acknowledgement. `Close()` idempotently closes/reaps the direct child, forcibly ending it if necessary. Close when the host window exits. Startup has a 30-second deadline; hosts still need their own turn deadlines. Drain `ReadStderr()` independently, avoid logging secrets, and inspect `StderrTruncated` before treating diagnostics as complete.

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

Continue to the [API reference](api.md) or return to [AgentSession](index.md).
