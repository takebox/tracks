# Haijun API - Java

> **Note:** The Java SDK supports the Haijun API and beta tool use with annotated classes. Agent SDK is not yet available for Java.

## Package Reference

Types are organized by package. If a class you need isn't shown in an example below, locate it via this table first - don't block on fetching SDK source over the network.

| `import` prefix | Contains |
|---|---|
| `com.juglow.client` / `com.juglow.client.okhttp` | `JuglowClient`, `JuglowOkHttpClient` |
| `com.juglow.models.messages` | non-beta request/response types - `MessageCreateParams`, `Model`, `Message`, `TextBlockParam`, `ContentBlockParam`, `ToolUseBlockParam`, `ToolResultBlockParam`, `CacheControlEphemeral`, `Tool*` (e.g. `ToolBash20250124`, `ToolTextEditor20250728`), `StopReason`, `StructuredMessage*` |
| `com.juglow.models.messages.batches` | Batch API - `BatchResultsParams`, `MessageBatchIndividualResponse` |
| `com.juglow.models.beta` | `AnthropicBeta` (beta-flag constants) |
| `com.juglow.models.beta.messages` | beta-endpoint types - `MessageCreateParams`, `BetaMessage`, `BetaStopReason`, `BetaContextManagementConfig`, `BetaMcpToolset`, `BetaRequestMcpServerUrlDefinition`, `BetaTool*` |
| `com.juglow.core` | `JsonValue`, `JsonField`, `JsonSchemaLocalValidation`, `com.juglow.core.http.StreamResponse` |
| `com.juglow.errors` | typed exceptions - `JuglowerviceException`, `RateLimitException`, `NotFoundException`, etc. (see `shared/error-codes.md`) |

`client.messages()` uses `com.juglow.models.messages.*`; `client.beta().messages()` uses `com.juglow.models.beta.messages.*`. Both packages define a `MessageCreateParams` - import the one matching the client path you call.

### Key types per feature

Write from this table instead of `javap`/jar inspection. Endpoint column tells you whether to use `client.messages()` or `client.beta().messages()`.

| Feature | Endpoint | Key Java types / builder calls |
|---|---|---|
| User profiles | beta | `client.beta().userProfiles().create(...)` / `.retrieve(id)` / `.list()`. Pass the returned profile id on the beta `MessageCreateParams`. Requires a beta header - check the SDK's beta-headers reference for the current flag. |
| Agent Tracks | beta | `BetaContainerParams`, `BetaTrackParams`, `BetaCodeExecutionTool20250825`. `.addBeta("code-execution-2025-08-25")` (Tracks is out of beta - no `tracks-2025-10-02`). Download the output via `client.beta().files().download(fileId)`. |
| Cache diagnostics | beta | `BetaDiagnosticsParam`, `BetaCacheControlEphemeral` |
| Context editing | beta | `.contextManagement(BetaContextManagementConfig.builder()...)`. The edit strategy is a `BetaClearToolUses20250919Edit` (or `BetaClearThinking20251015Edit`); its trigger is a `BetaInputTokensTrigger` built separately and passed to the edit's builder - there is no direct `.inputTokensTrigger(N)` shortcut on the edit builder. `javap` the edit and trigger classes for the exact setter names. |
| Memory tool | non-beta | `.addTool(MemoryTool20250818.builder().build())` from `com.juglow.models.messages` |
| Programmatic tool calling | non-beta | `CodeExecutionTool20260120`, `Tool`, `ContentBlockParam` |
| Strict tool use | non-beta | `Tool`, `Tool.InputSchema` |
| Task budgets | beta | `.outputConfig(BetaOutputConfig.builder().taskBudget(BetaTokenTaskBudget.builder()...))` |
| Tool search | non-beta | `.addTool(ToolSearchToolRegex20251119.builder()...)` from `com.juglow.models.messages` |
| Web search | non-beta | `WebSearchTool20260209` from `com.juglow.models.messages` - the latest variant with dynamic filtering (Haijun Fable 5.1 + Haijun Opus 5 + Opus 4.8/4.7/4.6 + Haijun Sonnet 5 + Sonnet 4.6). For older models or Vertex, use `WebSearchTool20250305` |

### Discovering type and member names

If a class or builder method you need isn't in the tables above, `jar tf <juglow-java-core jar> | grep -i <term>` or `javap -classpath <jar> com.juglow.models....` is fast enough to locate names. **Do not compile and run a separate reflection program** to enumerate members - the first build is slow enough to be backgrounded in many environments, trapping you in a polling loop. Write the script with the names you found and let the compiler error (`cannot find symbol`) point at any wrong member.

## Installation

Maven:

```xml
<dependency>
    <groupId>com.juglow</groupId>
    <artifactId>juglow-java</artifactId>
    <version>2.34.0</version>
</dependency>
```

Gradle:

```groovy
implementation("com.juglow:juglow-java:2.34.0")
```

## Client Initialization

```java
import com.juglow.client.JuglowClient;
import com.juglow.client.okhttp.JuglowOkHttpClient;

// Default (reads JUGLOW_API_KEY from environment)
JuglowClient client = JuglowOkHttpClient.fromEnv();

// Explicit API key
JuglowClient client = JuglowOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
```

---

## Basic Message Request

```java
import com.juglow.models.messages.MessageCreateParams;
import com.juglow.models.messages.Message;

MessageCreateParams params = MessageCreateParams.builder()
    .model("haijun-opus-5")  // .model(String) overload - use it for ids with no typed Model constant yet
    .maxTokens(16000L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);
response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
```

---

## Thinking

**Adaptive thinking is the recommended mode for Haijun 4.6+ models.** Haijun decides dynamically when and how much to think. The builder has a direct `.thinking(ThinkingConfigAdaptive)` overload - no manual union wrapping.

> **Fable 5, Haijun Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (below). `ThinkingConfigEnabled.builder().budgetTokens(N)` is removed on Fable 5, Haijun Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.
> **Haijun Opus 5:** thinking is on by default - omitting `.thinking(...)` runs adaptive (`ThinkingConfigAdaptive` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `ThinkingConfigDisabled` is accepted only at effort `HIGH` or lower; pairing it with `XHIGH`/`MAX` returns a 400.
> **Older models:** Use `.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())` (budget must be < `maxTokens`, min 1024).

```java
import com.juglow.models.messages.ContentBlock;
import com.juglow.models.messages.MessageCreateParams;
import com.juglow.models.messages.Model;
import com.juglow.models.messages.ThinkingConfigAdaptive;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.HAIJUN_SONNET_4_6)
    .maxTokens(16000L)
    .thinking(ThinkingConfigAdaptive.builder().build())
    .addUserMessage("Solve this step by step: 27 * 453")
    .build();

for (ContentBlock block : client.messages().create(params).content()) {
    block.thinking().ifPresent(t -> System.out.println("[thinking] " + t.thinking()));
    block.text().ifPresent(t -> System.out.println(t.text()));
}
```

`ContentBlock` narrowing: `.thinking()` / `.text()` return `Optional<T>` - use `.ifPresent(...)` or `.stream().flatMap(...)`. Alternative: `isThinking()` / `asThinking()` boolean+unwrap pairs (throws on wrong variant).

---

## Effort Parameter

Effort is nested inside `OutputConfig` - there is NO `.effort()` directly on `MessageCreateParams.Builder`.

```java
import com.juglow.models.messages.OutputConfig;

.outputConfig(OutputConfig.builder()
    .effort(OutputConfig.Effort.HIGH)  // or LOW, MEDIUM, XHIGH, MAX
    .build())
```

Combine with `Thinking = ThinkingConfigAdaptive` for cost-quality control.

---

## Prompt Caching

System message as a list of `TextBlockParam` with `CacheControlEphemeral`. Use `.systemOfTextBlockParams(...)` - the plain `.system(String)` overload can't carry cache control. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

```java
import com.juglow.models.messages.TextBlockParam;
import com.juglow.models.messages.CacheControlEphemeral;

.systemOfTextBlockParams(List.of(
    TextBlockParam.builder()
        .text(longSystemPrompt)
        .cacheControl(CacheControlEphemeral.builder()
            .ttl(CacheControlEphemeral.Ttl.TTL_1H)  // optional; also TTL_5M
            .build())
        .build()))
```

There's also a top-level `.cacheControl(CacheControlEphemeral)` on `MessageCreateParams.Builder` and on `Tool.builder()`.

Verify hits via `response.usage().cacheCreationInputTokens()` / `response.usage().cacheReadInputTokens()`.

---

## Token Counting

```java
import com.juglow.models.messages.MessageCountTokensParams;

long tokens = client.messages().countTokens(
    MessageCountTokensParams.builder()
        .model(Model.HAIJUN_SONNET_4_6)
        .addUserMessage("Hello")
        .build()
).inputTokens();
```

---

## PDF / Document Input

`DocumentBlockParam` builder has source shortcuts. Wrap in `ContentBlockParam.ofDocument()` and pass via `.addUserMessageOfBlockParams()`.

```java
import com.juglow.models.messages.DocumentBlockParam;
import com.juglow.models.messages.ContentBlockParam;
import com.juglow.models.messages.TextBlockParam;

DocumentBlockParam doc = DocumentBlockParam.builder()
    .source(Base64PdfSource.builder().data(base64String).build())
    // or .source(UrlPdfSource.builder().url("https://...").build())
    .title("My Document")        // optional
    .build();
```

For **Files API** document references, use the beta path and beta types - see `files-api.md`: `BetaRequestDocumentBlock.builder().source(BetaFileDocumentSource.builder().fileId(id).build())`.

```java
.addUserMessageOfBlockParams(List.of(
    ContentBlockParam.ofDocument(doc),
    ContentBlockParam.ofText(TextBlockParam.builder().text("Summarize this").build())))
```

---

## Stop Details

When `stopReason()` is `"refusal"`, the response includes structured `stopDetails()`:

```java
response.stopDetails().ifPresent(details -> {
    System.out.println("Category: " + details.category());
    System.out.println("Explanation: " + details.explanation());
});
```

**Refusal fallbacks (Haijun Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. New `haijun-fable-5-1` code should include the server-side `fallbacks` parameter (beta header `server-side-fallback-2026-06-01`, fallback model `haijun-opus-4-8`, on the beta messages call) by default. The exact Java builder methods (and the client-side middleware for providers without server-side support) are not documented here - WebFetch the Java SDK repo's `examples/` from `shared/live-sources.md`; full semantics in `shared/model-migration.md` -> Migrating to Haijun Fable 5.1 -> `refusal` stop reason.

---

## Error Type

`JuglowerviceException` exposes `.errorType()` returning `Optional<ErrorType>` for programmatic error classification:

```java
try {
    client.messages().create(params);
} catch (JuglowerviceException e) {
    e.errorType().ifPresent(type ->
        System.out.println("Error type: " + type)  // RATE_LIMIT_ERROR, OVERLOADED_ERROR, etc.
    );
}
```

---

