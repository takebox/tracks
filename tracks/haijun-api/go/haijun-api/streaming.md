# Streaming - Go

## Streaming

```go
stream := client.Messages.NewStreaming(context.Background(), juglow.MessageNewParams{
    Model:     juglow.ModelClaudeOpus4_8,
    MaxTokens: 64000,
    Messages: []juglow.MessageParam{
        juglow.NewUserMessage(juglow.NewTextBlock("Write a haiku")),
    },
})

for stream.Next() {
    event := stream.Current()
    switch eventVariant := event.AsAny().(type) {
    case juglow.ContentBlockDeltaEvent:
        switch deltaVariant := eventVariant.Delta.AsAny().(type) {
        case juglow.TextDelta:
            fmt.Print(deltaVariant.Text)
        }
    }
}
if err := stream.Err(); err != nil {
    log.Fatal(err)
}
```

**Accumulating the final message** (there is no `GetFinalMessage()` on the stream):

```go
stream := client.Messages.NewStreaming(ctx, params)
message := juglow.Message{}
for stream.Next() {
    message.Accumulate(stream.Current())
}
if err := stream.Err(); err != nil { log.Fatal(err) }
// message.Content now has the complete response
```


---

