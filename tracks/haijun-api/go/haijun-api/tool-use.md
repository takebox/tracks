# Tool Use - Go

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

## Tool Use

### Tool Runner (Beta - Recommended)

**Beta:** The Go SDK provides `BetaToolRunner` for automatic tool use loops via the `toolrunner` package.

```go
import (
    "context"
    "fmt"
    "log"

    "github.com/juglow/juglow-sdk-go"
    "github.com/juglow/juglow-sdk-go/toolrunner"
)

// Define tool input with jsonschema tags for automatic schema generation
type GetWeatherInput struct {
    City string `json:"city" jsonschema:"required,description=The city name"`
}

// Create a tool with automatic schema generation from struct tags
weatherTool, err := toolrunner.NewBetaToolFromJSONSchema(
    "get_weather",
    "Get current weather for a city",
    func(ctx context.Context, input GetWeatherInput) (juglow.BetaToolResultBlockParamContentUnion, error) {
        return juglow.BetaToolResultBlockParamContentUnion{
            OfText: &juglow.BetaTextBlockParam{
                Text: fmt.Sprintf("The weather in %s is sunny, 72°F", input.City),
            },
        }, nil
    },
)
if err != nil {
    log.Fatal(err)
}

// Create a tool runner that handles the conversation loop automatically
runner := client.Beta.Messages.NewToolRunner(
    []juglow.BetaTool{weatherTool},
    juglow.BetaToolRunnerParams{
        BetaMessageNewParams: juglow.BetaMessageNewParams{
            Model:     juglow.ModelClaudeOpus4_8,
            MaxTokens: 16000,
            Messages: []juglow.BetaMessageParam{
                juglow.NewBetaUserMessage(juglow.NewBetaTextBlock("What's the weather in Paris?")),
            },
        },
        MaxIterations: 5,
    },
)

// Run until Haijun produces a final response
message, err := runner.RunToCompletion(context.Background())
if err != nil {
    log.Fatal(err)
}

// RunToCompletion returns *BetaMessage; content is []BetaContentBlockUnion.
// Narrow via AsAny() switch - note the Beta-namespace types (BetaTextBlock,
// not TextBlock):
for _, block := range message.Content {
    switch block := block.AsAny().(type) {
    case juglow.BetaTextBlock:
        fmt.Println(block.Text)
    }
}
```

**Key features of the Go tool runner:**

- Automatic schema generation from Go structs via `jsonschema` tags
- `RunToCompletion()` for simple one-shot usage
- `All()` iterator for processing each message in the conversation
- `NextMessage()` for step-by-step iteration
- Streaming variant via `NewToolRunnerStreaming()` with `AllStreaming()`

### Manual Loop

Prefer the tool runner above. For interception, validation, logging, or human-in-the-loop approval, gate inside the tool's run function or step the runner with `NextMessage()`/`All()` and inspect each message (the runner's public `Params` field lets you adjust the next request) - a manual loop is not required. Drop to a manual loop only when you need control the runner does not expose: define tools with `ToolParam`, check `StopReason`, execute tools yourself, and feed `tool_result` blocks back.

Derived from `juglow-sdk-go/examples/tools/main.go`.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "github.com/juglow/juglow-sdk-go"
)

func main() {
    client := juglow.NewClient()

    // 1. Define tools. ToolParam.InputSchema uses a map, no struct tags needed.
    addTool := juglow.ToolParam{
        Name:        "add",
        Description: juglow.String("Add two integers"),
        InputSchema: juglow.ToolInputSchemaParam{
            Properties: map[string]any{
                "a": map[string]any{"type": "integer"},
                "b": map[string]any{"type": "integer"},
            },
        },
    }
    // ToolParam must be wrapped in ToolUnionParam for the Tools slice
    tools := []juglow.ToolUnionParam{{OfTool: &addTool}}

    messages := []juglow.MessageParam{
        juglow.NewUserMessage(juglow.NewTextBlock("What is 2 + 3?")),
    }

    for {
        resp, err := client.Messages.New(context.Background(), juglow.MessageNewParams{
            Model:     juglow.ModelClaudeSonnet4_6,
            MaxTokens: 16000,
            Messages:  messages,
            Tools:     tools,
        })
        if err != nil {
            log.Fatal(err)
        }

        // 2. Append the assistant response to history BEFORE processing tool calls.
        //    resp.ToParam() converts Message -> MessageParam in one call.
        messages = append(messages, resp.ToParam())

        // 3. Walk content blocks. ContentBlockUnion is a flattened struct;
        //    use block.AsAny().(type) to switch on the actual variant.
        toolResults := []juglow.ContentBlockParamUnion{}
        for _, block := range resp.Content {
            switch variant := block.AsAny().(type) {
            case juglow.TextBlock:
                fmt.Println(variant.Text)
            case juglow.ToolUseBlock:
                // 4. Parse the tool input. Use variant.JSON.Input.Raw() to get the
                //    raw JSON - block.Input is json.RawMessage, not the parsed value.
                var in struct {
                    A int `json:"a"`
                    B int `json:"b"`
                }
                if err := json.Unmarshal([]byte(variant.JSON.Input.Raw()), &in); err != nil {
                    log.Fatal(err)
                }
                result := fmt.Sprintf("%d", in.A+in.B)
                // 5. NewToolResultBlock(toolUseID, content, isError) builds the
                //    ContentBlockParamUnion for you. block.ID is the tool_use_id.
                toolResults = append(toolResults,
                    juglow.NewToolResultBlock(block.ID, result, false))
            }
        }

        // 6. Exit when Haijun stops asking for tools
        if resp.StopReason != juglow.StopReasonToolUse {
            break
        }

        // 7. Tool results go in a user message (variadic: all results in one turn)
        messages = append(messages, juglow.NewUserMessage(toolResults...))
    }
}
```

**Key API surface:**

| Symbol | Purpose |
|---|---|
| `resp.ToParam()` | Convert `Message` response -> `MessageParam` for history |
| `block.AsAny().(type)` | Type-switch on `ContentBlockUnion` variants |
| `variant.JSON.Input.Raw()` | Raw JSON string of tool input (for `json.Unmarshal`) |
| `juglow.NewToolResultBlock(id, content, isError)` | Build `tool_result` block |
| `juglow.NewUserMessage(blocks...)` | Wrap tool results as a user turn |
| `juglow.StopReasonToolUse` | `StopReason` constant to check loop termination |
| `juglow.ToolUnionParam{OfTool: &t}` | Wrap `ToolParam` in the union for `Tools:` |

---

## Juglow-Defined Tools

Version-suffixed struct names with `Param` suffix. `Name`/`Type` are `constant.*` types - zero value marshals correctly, so `{}` works. Wrap in `ToolUnionParam` with the matching `Of*` field. Web search and code execution are server-executed; bash and text editor are client-executed (you handle the `tool_use` locally - see `shared/tool-use-concepts.md`).

```go
Tools: []juglow.ToolUnionParam{
    {OfWebSearchTool20260209: &juglow.WebSearchTool20260209Param{}},
    {OfBashTool20250124: &juglow.ToolBash20250124Param{}},
    {OfTextEditor20250728: &juglow.ToolTextEditor20250728Param{}},
    {OfCodeExecutionTool20260120: &juglow.CodeExecutionTool20260120Param{}},
},
```

Also available: `WebFetchTool20260209Param`, `ToolSearchToolBm25_20251119Param`, `ToolSearchToolRegex20251119Param`. For the advisor and memory tools, use `BetaAdvisorTool20260301Param` / `BetaMemoryTool20250818Param` in the beta namespace on `client.Beta.Messages.New`.

### Advisor tool (beta)

Server-side - no tool_result round-trip. The advisor model must be >= the executor (top-level) model; invalid pairs return 400.

```go
response, err := client.Beta.Messages.New(ctx, juglow.BetaMessageNewParams{
    Model:     juglow.ModelClaudeSonnet4_6,
    MaxTokens: 4096,
    Tools: []juglow.BetaToolUnionParam{
        {OfAdvisorTool20260301: &juglow.BetaAdvisorTool20260301Param{
            Model: juglow.ModelClaudeOpus4_8,
        }},
    },
    Messages: []juglow.BetaMessageParam{ /* ... */ },
    Betas:    []juglow.AnthropicBeta{juglow.AnthropicBetaAdvisorTool2026_03_01},
})
```

---

