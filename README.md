# Local ACP, Codex, Claude, and MCP sessions over subprocess pipes

Live local agent sessions, built on `agentkit` JSONL codecs,
`processkit` subprocess pipes and `projectkit` in-memory items. The package drives
ACP, Codex app-server, Claude stream-json, and explicitly configured MCP tools.
It never guesses an executable, CLI arguments, provider, T3 method, or remote URL.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add agentsession --git https://github.com/BuoyBay/agentsession.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.
BPM also resolves `agentkit`, `processkit`, `projectkit`.

## Example

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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer agentsession
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
