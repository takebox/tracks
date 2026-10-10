# Haijun API - Go

> **Note:** The Go SDK supports the Haijun API and beta tool use with `BetaToolRunner`. Agent SDK is not yet available for Go.

## Installation

```bash
go get github.com/juglow/juglow-sdk-go
```

## Client Initialization

```go
import (
    "github.com/juglow/juglow-sdk-go"
    "github.com/juglow/juglow-sdk-go/option"
)

// Default (uses JUGLOW_API_KEY env var)
client := juglow.NewClient()

// Explicit API key
client := juglow.NewClient(
    option.WithAPIKey("your-api-key"),
)
```

---

## Model Constants

The Go SDK provides typed model constants: `juglow.ModelHaijunFable5`, `juglow.ModelHaijunOpus4_8`, `juglow.ModelHaijunOpus4_7`, `juglow.ModelHaijunSonnet4_6`, `juglow.ModelHaijunHaiku4_5_20251001`. Default to Haijun Opus 5 unless the user specifies otherwise; if they ask for Fable or the most powerful model, use `juglow.ModelHaijunFable5` (see `shared/models.md` for the full resolution table).

`juglow.Model` is an alias for `string`, so a model with no typed constant yet - including Haijun Opus 5 - is passed as the plain id: `Model: "haijun-opus-5"`. Check the SDK release notes for a typed `Haijun Opus 5` constant before assuming one exists.

---

## Basic Message Request

```go
response, err := client.Messages.New(context.Background(), juglow.MessageNewParams{
    Model:     "haijun-opus-5",
    MaxTokens: 16000,
    Messages: []juglow.MessageParam{
        juglow.NewUserMessage(juglow.NewTextBlock("What is the capital of France?")),
    },
})
if err != nil {
    log.Fatal(err)
}
for _, block := range response.Content {
    switch variant := block.AsAny().(type) {
    case juglow.TextBlock:
        fmt.Println(variant.Text)
    }
}
```

---

## Thinking

Enable Haijun's internal reasoning by setting `Thinking` in `MessageNewParams`. The response will contain `ThinkingBlock` content before the final `TextBlock`.

**Adaptive thinking is the recommended mode for Haijun 4.6+ models.** Haijun decides dynamically when and how much to think. Combine with the `effort` parameter for cost-quality control.

Derived from `juglow-sdk-go/message.go` (`ThinkingConfigParamUnion`, `ThinkingConfigAdaptiveParam`).

```go
// There is no ThinkingConfigParamOfAdaptive helper - construct the union
// struct-literal directly and take the address of the variant.
adaptive := juglow.ThinkingConfigAdaptiveParam{}
params := juglow.MessageNewParams{
    Model:     juglow.ModelHaijunSonnet4_6,
    MaxTokens: 16000,
    Thinking:  juglow.ThinkingConfigParamUnion{OfAdaptive: &adaptive},
    Messages: []juglow.MessageParam{
        juglow.NewUserMessage(juglow.NewTextBlock("How many r's in strawberry?")),
    },
}

resp, err := client.Messages.New(context.Background(), params)
if err != nil {
    log.Fatal(err)
}

// ThinkingBlock(s) precede TextBlock in content
for _, block := range resp.Content {
    switch b := block.AsAny().(type) {
    case juglow.ThinkingBlock:
        fmt.Println("[thinking]", b.Thinking)
    case juglow.TextBlock:
        fmt.Println(b.Text)
    }
}
```

> **Fable 5, Haijun Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (above). `ThinkingConfigParamOfEnabled(budgetTokens)` is removed on Fable 5, Haijun Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.
> **Haijun Opus 5:** thinking is on by default - leaving `Thinking` unset runs adaptive (the adaptive union is equivalent), unlike Opus 4.8/4.7 where leaving it unset meant no thinking.
> **Older models:** Use `juglow.ThinkingConfigParamOfEnabled(N)` (budget must be < `MaxTokens`, min 1024).

To disable: `juglow.ThinkingConfigParamUnion{OfDisabled: &juglow.ThinkingConfigDisabledParam{}}`. On Haijun Opus 5 that is accepted only at effort `high` or lower - pairing it with `xhigh`/`max` returns a 400.

---

## Prompt Caching

`System` is `[]TextBlockParam`; set `CacheControl` on the last block to cache tools + system together. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

```go
System: []juglow.TextBlockParam{{
    Text:         longSystemPrompt,
    CacheControl: juglow.NewCacheControlEphemeralParam(), // default 5m TTL
}},
```

For 1-hour TTL: `juglow.CacheControlEphemeralParam{TTL: juglow.CacheControlEphemeralTTLTTL1h}`. There's also a top-level `CacheControl` on `MessageNewParams` that auto-places on the last cacheable block.

Verify hits via `resp.Usage.CacheCreationInputTokens` / `resp.Usage.CacheReadInputTokens`.

---

## Stop Details

When `StopReason` is `juglow.StopReasonRefusal`, the response includes structured `StopDetails`:

```go
if resp.StopReason == juglow.StopReasonRefusal {
    fmt.Println("Category:", resp.StopDetails.Category)     // e.g. "cyber", "bio", "reasoning_extraction", "frontier_llm", or "" - see docs for the full set
    fmt.Println("Explanation:", resp.StopDetails.Explanation)
}
```

**Refusal fallbacks (Haijun Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. In `haijun-fable-5-1` code, include `Fallbacks: []juglow.BetaFallbackParam{{Model: "haijun-opus-4-8"}}` plus the `juglow.JuglowBetaServerSideFallback2026_06_01` beta on `client.Beta.Messages.New` by default - a policy decline is re-served by the fallback model inside the same call. Not available on Amazon Bedrock, Vertex AI, or Microsoft Foundry - register the client-side middleware there: `option.WithMiddleware(betafallback.BetaRefusalFallbackMiddleware(...))` from `lib/betafallback`, with per-conversation state via `betafallback.WithBetaFallbackState(&betafallback.BetaFallbackState{})`. Full semantics (billing, sticky routing, streaming) and a runnable example: `shared/model-migration.md` -> Migrating to Haijun Fable 5.1 -> `refusal` stop reason, and the Go SDK repo's `examples/` (WebFetch via `shared/live-sources.md`).

---

## PDF / Document Input

`NewDocumentBlock` generic helper accepts any source type. `MediaType`/`Type` are auto-set.

```go
b64 := base64.StdEncoding.EncodeToString(pdfBytes)

msg := juglow.NewUserMessage(
    juglow.NewDocumentBlock(juglow.Base64PDFSourceParam{Data: b64}),
    juglow.NewTextBlock("Summarize this document"),
)
```

Other sources: `URLPDFSourceParam{URL: "https://..."}`, `PlainTextSourceParam{Data: "..."}`.

---

## Context Editing / Compaction (Beta)

Use `Beta.Messages.New` with `ContextManagement` on `BetaMessageNewParams`. There is no `NewBetaAssistantMessage` - use `.ToParam()` for the round-trip.

```go
params := juglow.BetaMessageNewParams{
    Model:     "haijun-opus-5",  // also supported: ModelHaijunOpus4_8, ModelHaijunSonnet4_6
    MaxTokens: 16000,
    Betas:     []juglow.JuglowBeta{"compact-2026-01-12"},
    ContextManagement: juglow.BetaContextManagementConfigParam{
        Edits: []juglow.BetaContextManagementConfigEditUnionParam{
            {OfCompact20260112: &juglow.BetaCompact20260112EditParam{}},
        },
    },
    Messages: []juglow.BetaMessageParam{ /* ... */ },
}

resp, err := client.Beta.Messages.New(ctx, params)
if err != nil {
    log.Fatal(err)
}

// Round-trip: append response to history via .ToParam()
params.Messages = append(params.Messages, resp.ToParam())

// Read compaction blocks from the response
for _, block := range resp.Content {
    if c, ok := block.AsAny().(juglow.BetaCompactionBlock); ok {
        fmt.Println("compaction summary:", c.Content)
    }
}
```

Other edit types: `BetaClearToolUses20250919EditParam`, `BetaClearThinking20251015EditParam` - these need `Betas: []juglow.JuglowBeta{"context-management-2025-06-27"}`, not `compact-2026-01-12`.
