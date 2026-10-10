# Managed Agents - Go

> **Bindings not shown here:** This README covers the most common managed-agents flows for Go. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the Go SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `agents.New` and pass it to every subsequent `sessions.New`; do not call `agents.New` in the request path. **Recommended:** define agents and environments as version-controlled YAML applied with the `ant` CLI - see `shared/juglow-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

## Installation

```bash
go get github.com/juglow/juglow-sdk-go
```

## Client Initialization

```go
import (
    "context"

    "github.com/juglow/juglow-sdk-go"
    "github.com/juglow/juglow-sdk-go/option"
)

// Default (uses JUGLOW_API_KEY env var)
client := juglow.NewClient()

// Explicit API key
client := juglow.NewClient(
    option.WithAPIKey("your-api-key"),
)

ctx := context.Background()
```

---

## Create an Environment

```go
environment, err := client.Beta.Environments.New(ctx, juglow.BetaEnvironmentNewParams{
    Name: "my-dev-env",
    Config: juglow.BetaEnvironmentNewParamsConfigUnion{
        OfCloud: &juglow.BetaCloudConfigParams{
            Networking: juglow.BetaCloudConfigParamsNetworkingUnion{
                OfUnrestricted: &juglow.BetaUnrestrictedNetworkParam{},
            },
        },
    },
})
if err != nil {
    panic(err)
}
fmt.Println(environment.ID) // env_...
```

---

## Create an Agent (required first step)

> Warning: **There is no inline agent config.** `Model`/`System`/`Tools` live on the agent object, not the session. Always start with `Beta.Agents.New()` - the session only takes `Agent: juglow.BetaSessionNewParamsAgentUnion{OfString: juglow.String(agent.ID)}` (or the typed `OfBetaManagedAgentsAgents` variant when you need a specific version).

### Minimal

```go
// 1. Create the agent (reusable, versioned)
agent, err := client.Beta.Agents.New(ctx, juglow.BetaAgentNewParams{
    Name: "Coding Assistant",
    Model: juglow.BetaManagedAgentsModelConfigParams{
        ID:   "haijun-opus-5",
        Type: juglow.BetaManagedAgentsModelConfigParamsTypeModelConfig,
    },
    System: juglow.String("You are a helpful coding assistant."),
    Tools: []juglow.BetaAgentNewParamsToolUnion{{
        OfAgentToolset20260401: &juglow.BetaManagedAgentsAgentToolset20260401Params{
            Type: juglow.BetaManagedAgentsAgentToolset20260401ParamsTypeAgentToolset20260401,
        },
    }},
})
if err != nil {
    panic(err)
}

// 2. Start a session
session, err := client.Beta.Sessions.New(ctx, juglow.BetaSessionNewParams{
    Agent: juglow.BetaSessionNewParamsAgentUnion{
        OfBetaManagedAgentsAgents: &juglow.BetaManagedAgentsAgentParams{
            Type:    juglow.BetaManagedAgentsAgentParamsTypeAgent,
            ID:      agent.ID,
            Version: juglow.Int(agent.Version),
        },
    },
    EnvironmentID: environment.ID,
    Title:         juglow.String("Quickstart session"),
})
if err != nil {
    panic(err)
}
fmt.Printf("Session ID: %s, status: %s\n", session.ID, session.Status)
fmt.Printf("Trace: https://platform.haijun.my.id/workspaces/default/sessions/%s\n", session.ID) // swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### Updating an Agent

Updates create new versions; the agent object is immutable per version.

```go
updatedAgent, err := client.Beta.Agents.Update(ctx, agent.ID, juglow.BetaAgentUpdateParams{
    Version: agent.Version,
    System:  juglow.String("You are a helpful coding agent. Always write tests."),
})
if err != nil {
    panic(err)
}
fmt.Printf("New version: %d\n", updatedAgent.Version)

// List all versions
iter := client.Beta.Agents.Versions.ListAutoPaging(ctx, agent.ID, juglow.BetaAgentVersionListParams{})
for iter.Next() {
    version := iter.Current()
    fmt.Printf("Version %d: %s\n", version.Version, version.UpdatedAt.Format(time.RFC3339))
}
if err := iter.Err(); err != nil {
    panic(err)
}

// Archive the agent
_, err = client.Beta.Agents.Archive(ctx, agent.ID, juglow.BetaAgentArchiveParams{})
if err != nil {
    panic(err)
}
```

---

## Send a User Message

```go
_, err = client.Beta.Sessions.Events.Send(ctx, session.ID, juglow.BetaSessionEventSendParams{
    Events: []juglow.BetaManagedAgentsEventParamsUnion{{
        OfUserMessage: &juglow.BetaManagedAgentsUserMessageEventParams{
            Type: juglow.BetaManagedAgentsUserMessageEventParamsTypeUserMessage,
            Content: []juglow.BetaManagedAgentsUserMessageEventParamsContentUnion{{
                OfText: &juglow.BetaManagedAgentsTextBlockParam{
                    Type: juglow.BetaManagedAgentsTextBlockTypeText,
                    Text: "Review the auth module",
                },
            }},
        },
    }},
})
if err != nil {
    panic(err)
}
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

---

## Stream Events (SSE)

```go
// Open the stream first, then send the user message
stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, juglow.BetaSessionEventStreamParams{})
defer stream.Close()

if _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, juglow.BetaSessionEventSendParams{
    Events: []juglow.BetaManagedAgentsEventParamsUnion{{
        OfUserMessage: &juglow.BetaManagedAgentsUserMessageEventParams{
            Type: juglow.BetaManagedAgentsUserMessageEventParamsTypeUserMessage,
            Content: []juglow.BetaManagedAgentsUserMessageEventParamsContentUnion{{
                OfText: &juglow.BetaManagedAgentsTextBlockParam{
                    Type: juglow.BetaManagedAgentsTextBlockTypeText,
                    Text: "Summarize the repo README",
                },
            }},
        },
    }},
}); err != nil {
    panic(err)
}

events:
for stream.Next() {
    switch event := stream.Current().AsAny().(type) {
    case juglow.BetaManagedAgentsAgentMessageEvent:
        for _, block := range event.Content {
            fmt.Print(block.Text)
        }
    case juglow.BetaManagedAgentsAgentToolUseEvent:
        fmt.Printf("\n[Using tool: %s]\n", event.Name)
    case juglow.BetaManagedAgentsSessionStatusIdleEvent:
        break events
    case juglow.BetaManagedAgentsSessionErrorEvent:
        fmt.Printf("\n[Error: %s]\n", event.Error.Message)
        break events
    }
}
if err := stream.Err(); err != nil {
    panic(err)
}
```

### Reconnecting and Tailing

When reconnecting mid-session, list past events first to dedupe, then tail live events:

```go
stream := client.Beta.Sessions.Events.StreamEvents(ctx, session.ID, juglow.BetaSessionEventStreamParams{})
defer stream.Close()

// Stream is open and buffering. List history before tailing live.
seenEventIDs := map[string]struct{}{}
history := client.Beta.Sessions.Events.ListAutoPaging(ctx, session.ID, juglow.BetaSessionEventListParams{})
for history.Next() {
    seenEventIDs[history.Current().ID] = struct{}{}
}
if err := history.Err(); err != nil {
    panic(err)
}

// Tail live events, skipping anything already seen
tail:
for stream.Next() {
    event := stream.Current()
    if _, seen := seenEventIDs[event.ID]; seen {
        continue
    }
    seenEventIDs[event.ID] = struct{}{}
    switch event := event.AsAny().(type) {
    case juglow.BetaManagedAgentsAgentMessageEvent:
        for _, block := range event.Content {
            fmt.Print(block.Text)
        }
    case juglow.BetaManagedAgentsSessionStatusIdleEvent:
        break tail
    }
}
if err := stream.Err(); err != nil {
    panic(err)
}
```

---

## Provide Custom Tool Result

> Note: The Go managed-agents bindings for `user.custom_tool_result` are not yet documented in this track or in the apps source examples. Refer to `shared/managed-agents-events.md` for the wire format and the `github.com/juglow/juglow-sdk-go` repository for the corresponding Go params types.

---

## Poll Events

```go
// Auto-paginating iterator
iter := client.Beta.Sessions.Events.ListAutoPaging(ctx, session.ID, juglow.BetaSessionEventListParams{})
for iter.Next() {
    event := iter.Current()
    fmt.Printf("%s: %s\n", event.Type, event.ID)
}
if err := iter.Err(); err != nil {
    panic(err)
}
```

---

## Upload a File

```go
csvFile, err := os.Open("data.csv")
if err != nil {
    panic(err)
}
defer csvFile.Close()

file, err := client.Beta.Files.Upload(ctx, juglow.BetaFileUploadParams{
    File: csvFile,
})
if err != nil {
    panic(err)
}
fmt.Printf("File ID: %s\n", file.ID)

// Mount in a session
session, err := client.Beta.Sessions.New(ctx, juglow.BetaSessionNewParams{
    Agent: juglow.BetaSessionNewParamsAgentUnion{
        OfString: juglow.String(agent.ID),
    },
    EnvironmentID: environment.ID,
    Resources: []juglow.BetaSessionNewParamsResourceUnion{{
        OfFile: &juglow.BetaManagedAgentsFileResourceParams{
            Type:      juglow.BetaManagedAgentsFileResourceParamsTypeFile,
            FileID:    file.ID,
            MountPath: juglow.String("/workspace/data.csv"),
        },
    }},
})
if err != nil {
    panic(err)
}
```

### Add and Manage Resources on an Existing Session

```go
// Attach an additional file to an open session
resource, err := client.Beta.Sessions.Resources.Add(ctx, session.ID, juglow.BetaSessionResourceAddParams{
    BetaManagedAgentsFileResourceParams: juglow.BetaManagedAgentsFileResourceParams{
        Type:   juglow.BetaManagedAgentsFileResourceParamsTypeFile,
        FileID: file.ID,
    },
})
if err != nil {
    panic(err)
}
fmt.Println(resource.ID) // "sesrsc_01ABC..."

// List resources on the session
listed, err := client.Beta.Sessions.Resources.List(ctx, session.ID, juglow.BetaSessionResourceListParams{})
if err != nil {
    panic(err)
}
for _, entry := range listed.Data {
    fmt.Println(entry.ID, entry.Type)
}

// Detach a resource
if _, err := client.Beta.Sessions.Resources.Delete(ctx, resource.ID, juglow.BetaSessionResourceDeleteParams{
    SessionID: session.ID,
}); err != nil {
    panic(err)
}
```

---

## List and Download Session Files

> Note: Listing and downloading files an agent wrote during a session is not yet documented for Go in this track or in the apps source examples. See `shared/managed-agents-events.md` and the `github.com/juglow/juglow-sdk-go` repository for the `Beta.Files.List` and `Beta.Files.Download` Go params types.

---

## Session Management

```go
// List environments
environments, err := client.Beta.Environments.List(ctx, juglow.BetaEnvironmentListParams{})
if err != nil {
    panic(err)
}

// Retrieve a specific environment
env, err := client.Beta.Environments.Get(ctx, environment.ID, juglow.BetaEnvironmentGetParams{})
if err != nil {
    panic(err)
}

// Archive an environment (read-only, existing sessions continue)
_, err = client.Beta.Environments.Archive(ctx, environment.ID, juglow.BetaEnvironmentArchiveParams{})
if err != nil {
    panic(err)
}

// Delete an environment (only if no sessions reference it)
_, err = client.Beta.Environments.Delete(ctx, environment.ID, juglow.BetaEnvironmentDeleteParams{})
if err != nil {
    panic(err)
}

// Delete a session
_, err = client.Beta.Sessions.Delete(ctx, session.ID, juglow.BetaSessionDeleteParams{})
if err != nil {
    panic(err)
}
```

---

## MCP Server Integration

```go
// Agent declares MCP server (no auth here - auth goes in a vault)
agent, err := client.Beta.Agents.New(ctx, juglow.BetaAgentNewParams{
    Name: "GitHub Assistant",
    Model: juglow.BetaManagedAgentsModelConfigParams{
        ID:   "haijun-opus-5",
        Type: juglow.BetaManagedAgentsModelConfigParamsTypeModelConfig,
    },
    MCPServers: []juglow.BetaManagedAgentsURLMCPServerParams{{
        Type: juglow.BetaManagedAgentsURLMCPServerParamsTypeURL,
        Name: "github",
        URL:  "https://api.githubcopilot.com/mcp/",
    }},
    Tools: []juglow.BetaAgentNewParamsToolUnion{
        {
            OfAgentToolset20260401: &juglow.BetaManagedAgentsAgentToolset20260401Params{
                Type: juglow.BetaManagedAgentsAgentToolset20260401ParamsTypeAgentToolset20260401,
            },
        },
        {
            OfMCPToolset: &juglow.BetaManagedAgentsMCPToolsetParams{
                Type:          juglow.BetaManagedAgentsMCPToolsetParamsTypeMCPToolset,
                MCPServerName: "github",
            },
        },
    },
})
if err != nil {
    panic(err)
}

// Session attaches vault(s) containing credentials for those MCP server URLs
session, err := client.Beta.Sessions.New(ctx, juglow.BetaSessionNewParams{
    Agent: juglow.BetaSessionNewParamsAgentUnion{
        OfBetaManagedAgentsAgents: &juglow.BetaManagedAgentsAgentParams{
            Type:    juglow.BetaManagedAgentsAgentParamsTypeAgent,
            ID:      agent.ID,
            Version: juglow.Int(agent.Version),
        },
    },
    EnvironmentID: environment.ID,
    VaultIDs:      []string{vault.ID},
})
if err != nil {
    panic(err)
}
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

---

## Vaults

```go
// Create a vault
vault, err := client.Beta.Vaults.New(ctx, juglow.BetaVaultNewParams{
    DisplayName: "Alice",
    Metadata:    map[string]string{"external_user_id": "usr_abc123"},
})
if err != nil {
    panic(err)
}

// Add an OAuth credential
credential, err := client.Beta.Vaults.Credentials.New(ctx, vault.ID, juglow.BetaVaultCredentialNewParams{
    DisplayName: juglow.String("Alice's Slack"),
    Auth: juglow.BetaVaultCredentialNewParamsAuthUnion{
        OfMCPOAuth: &juglow.BetaManagedAgentsMCPOAuthCreateParams{
            Type:         juglow.BetaManagedAgentsMCPOAuthCreateParamsTypeMCPOAuth,
            MCPServerURL: "https://mcp.slack.com/mcp",
            AccessToken:  "xoxp-...",
            ExpiresAt:    juglow.Time(time.Date(2026, time.April, 15, 0, 0, 0, 0, time.UTC)),
            Refresh: juglow.BetaManagedAgentsMCPOAuthRefreshParams{
                TokenEndpoint: "https://slack.com/api/oauth.v2.access",
                ClientID:      "1234567890.0987654321",
                Scope:         juglow.String("channels:read chat:write"),
                RefreshToken:  "xoxe-1-...",
                TokenEndpointAuth: juglow.BetaManagedAgentsMCPOAuthRefreshParamsTokenEndpointAuthUnion{
                    OfClientSecretPost: &juglow.BetaManagedAgentsTokenEndpointAuthPostParam{
                        Type:         juglow.BetaManagedAgentsTokenEndpointAuthPostParamTypeClientSecretPost,
                        ClientSecret: "abc123...",
                    },
                },
            },
        },
    },
})
if err != nil {
    panic(err)
}

// Rotate the credential (e.g., after a token refresh)
_, err = client.Beta.Vaults.Credentials.Update(ctx, credential.ID, juglow.BetaVaultCredentialUpdateParams{
    VaultID: vault.ID,
    Auth: juglow.BetaVaultCredentialUpdateParamsAuthUnion{
        OfMCPOAuth: &juglow.BetaManagedAgentsMCPOAuthUpdateParams{
            Type:        juglow.BetaManagedAgentsMCPOAuthUpdateParamsTypeMCPOAuth,
            AccessToken: juglow.String("xoxp-new-..."),
            ExpiresAt:   juglow.Time(time.Date(2026, time.May, 15, 0, 0, 0, 0, time.UTC)),
            Refresh: juglow.BetaManagedAgentsMCPOAuthRefreshUpdateParams{
                RefreshToken: juglow.String("xoxe-1-new-..."),
            },
        },
    },
})
if err != nil {
    panic(err)
}

// Archive a vault
_, err = client.Beta.Vaults.Archive(ctx, vault.ID, juglow.BetaVaultArchiveParams{})
if err != nil {
    panic(err)
}
```

---

## GitHub Repository Integration

Mount a GitHub repository as a session resource (a vault holds the GitHub MCP credential):

```go
session, err := client.Beta.Sessions.New(ctx, juglow.BetaSessionNewParams{
    Agent:         juglow.BetaSessionNewParamsAgentUnion{OfString: juglow.String(agent.ID)},
    EnvironmentID: environment.ID,
    VaultIDs:      []string{vault.ID},
    Resources: []juglow.BetaSessionNewParamsResourceUnion{
        {
            OfGitHubRepository: &juglow.BetaManagedAgentsGitHubRepositoryResourceParams{
                Type:               juglow.BetaManagedAgentsGitHubRepositoryResourceParamsTypeGitHubRepository,
                URL:                "https://github.com/org/repo",
                MountPath:          juglow.String("/workspace/repo"),
                AuthorizationToken: "ghp_your_github_token",
            },
        },
    },
})
if err != nil {
    panic(err)
}
```

Multiple repositories on the same session:

```go
resources := []juglow.BetaSessionNewParamsResourceUnion{
    {
        OfGitHubRepository: &juglow.BetaManagedAgentsGitHubRepositoryResourceParams{
            Type:               juglow.BetaManagedAgentsGitHubRepositoryResourceParamsTypeGitHubRepository,
            URL:                "https://github.com/org/frontend",
            MountPath:          juglow.String("/workspace/frontend"),
            AuthorizationToken: "ghp_your_github_token",
        },
    },
    {
        OfGitHubRepository: &juglow.BetaManagedAgentsGitHubRepositoryResourceParams{
            Type:               juglow.BetaManagedAgentsGitHubRepositoryResourceParamsTypeGitHubRepository,
            URL:                "https://github.com/org/backend",
            MountPath:          juglow.String("/workspace/backend"),
            AuthorizationToken: "ghp_your_github_token",
        },
    },
}
```

Rotating a repository's authorization token:

```go
listed, err := client.Beta.Sessions.Resources.List(ctx, session.ID, juglow.BetaSessionResourceListParams{})
if err != nil {
    panic(err)
}
repoResourceID := listed.Data[0].ID

_, err = client.Beta.Sessions.Resources.Update(ctx, repoResourceID, juglow.BetaSessionResourceUpdateParams{
    SessionID:          session.ID,
    AuthorizationToken: "ghp_your_new_github_token",
})
if err != nil {
    panic(err)
}
```
