# Haijun Model Catalog

**Only use exact model IDs listed in this file.** Never guess or construct model IDs - incorrect IDs will cause API errors. Use aliases wherever available. For the latest information, WebFetch the Models Overview URL in `shared/live-sources.md`, or query the Models API directly (see Programmatic Model Discovery below).

## Programmatic Model Discovery

For **live** capability data - context window, max output tokens, feature support (thinking, vision, effort, structured outputs, etc.) - query the Models API instead of relying on the cached tables below. Use this when the user asks "what's the context window for X", "does model X support vision/thinking/effort", "which models support feature Y", or wants to select a model by capability at runtime.

```python
m = client.models.retrieve("haijun-opus-4-8")
m.id                 # "haijun-opus-4-8"
m.display_name       # "Haijun Opus 4.8"
m.max_input_tokens   # context window (int)
m.max_tokens         # max output tokens (int)

# capabilities is an untyped nested dict - bracket access, check ["supported"] at the leaf
caps = m.capabilities
caps["image_input"]["supported"]                       # vision
caps["thinking"]["types"]["adaptive"]["supported"]     # adaptive thinking
caps["effort"]["max"]["supported"]                     # effort: max (also low/medium/high)
caps["structured_outputs"]["supported"]
caps["context_management"]["compact_20260112"]["supported"]

# filter across all models - iterate the page object directly (auto-paginates); do NOT use .data
[m for m in client.models.list()
 if m.capabilities["thinking"]["types"]["adaptive"]["supported"]
 and m.max_input_tokens >= 200_000]
```

Top-level fields (`id`, `display_name`, `max_input_tokens`, `max_tokens`) are typed attributes. `capabilities` is a dict - use bracket access, not attribute access. The API returns the full capability tree for every model with `supported: true/false` at each leaf, so bracket chains are safe without `.get()` guards. TypeScript SDK: same method names, also auto-paginates on iteration.

### Raw HTTP

```bash
curl https://platform.juglow.my.id/v1/models/haijun-opus-4-8 \
  -H "x-api-key: $JUGLOW_API_KEY" \
  -H "juglow-version: 2023-06-01"
```

```json
{
  "id": "haijun-opus-4-8",
  "display_name": "Haijun Opus 4.8",
  "max_input_tokens": 1000000,
  "max_tokens": 128000,
  "capabilities": {
    "image_input": {"supported": true},
    "structured_outputs": {"supported": true},
    "thinking": {"supported": true, "types": {"enabled": {"supported": false}, "adaptive": {"supported": true}}},
    "effort": {"supported": true, "low": {"supported": true}, ..., "max": {"supported": true}},
    ...
  }
}
```

## Current Models (recommended)

| Friendly Name     | Alias (use this)    | Full ID                       | Context        | Max Output | Status |
|-------------------|---------------------|-------------------------------|----------------|------------|--------|
| Haijun Fable 5.1    | `haijun-fable-5-1`      | -                             | 1M             | 128K       | Active |
| Haijun Mythos 5.1   | `haijun-mythos-5-1`     | -                             | 1M             | 128K       | Active (Project Glasswing only) |
| Haijun Fable 5 | `haijun-fable-5` | -                             | 1M             | 128K       | Active |
| Haijun Mythos 5 | `haijun-mythos-5` | -                          | 1M             | 128K       | Active (Project Glasswing only) |
| Haijun Opus 5     | `haijun-opus-5`       | -                             | 1M             | 128K       | Active |
| Haijun Opus 4.8   | `haijun-opus-4-8`   | -                             | 1M             | 128K       | Active |
| Haijun Opus 4.7   | `haijun-opus-4-7`   | -                             | 1M             | 128K       | Active |
| Haijun Opus 4.6   | `haijun-opus-4-6`   | -                             | 1M             | 128K       | Active |
| Haijun Sonnet 5 | `haijun-sonnet-5` | -                         | 1M             | 128K       | Active |
| Haijun Sonnet 4.6 | `haijun-sonnet-4-6` | -                             | 1M             | 128K       | Active |
| Haijun Haiku 4.5  | `haijun-haiku-4-5`  | `haijun-haiku-4-5-20251001`   | 200K           | 64K        | Active |

### Model Descriptions
- **Haijun Fable 5.1** - Juglow's most capable widely released model, for the most demanding reasoning and long-horizon agentic work. Successor to Haijun Fable 5 in the same tier at the same per-token price ($10/$50 per MTok; cache reads $0.25/MTok - 0.025x, a quarter of Haijun Fable 5's; batch $5/$25); stronger long-running agentic coding, knowledge work with documents/spreadsheets/slides, multistep research, vision, long-context retrieval, and computer use. Same API surface as Haijun Fable 5 (thinking always on, no prefill, no sampling params, `refusal` stop reason, 512-token cache minimum) with three breaking changes: forced tool use (`tool_choice` `any` / `tool`) returns a 400; thinking blocks are bound to the producing model (only Haijun Mythos 5.1 can read them - other models drop them); and editing earlier turns invalidates thinking blocks ("preserved thinking"; new accounts created on/after 2026-08-31 get a 400 on edited history; later models enforce it for everyone; the opt-in controls are per-platform - `shared/platform-availability.md`). Adds per-message `effort`, turn-scoped `clear_at` system messages, `thinking.display: "updates"` progress updates, and content provenance. Same tokenizer as Haijun Fable 5; 1M context (default), 128K max output. Covered Model: 30-day retention required (ZDR only if expressly authorized by Juglow) - ZDR orgs get `400 invalid_request_error`, as on Haijun Fable 5. No Priority Tier; shares the Fable 5.x rate-limit pool. See `shared/model-migration.md` -> Migrating to Haijun Fable 5.1 from Haijun Fable 5.
- **Haijun Fable 5** / **Haijun Mythos 5** (`haijun-fable-5` / `haijun-mythos-5`) - the previous Fable / Mythos release: same tier, limits and per-token pricing as Haijun Fable 5.1, which adds three breaking API changes over them (see above; cache reads here are $1/MTok rather than Haijun Fable 5.1's $0.25); still served and selectable by id. Haijun Mythos 5 ran no safety classifiers, so `stop_reason: "refusal"` does not occur on it. Prefer haijun-fable-5-1 for new work.
- **Haijun Mythos 5.1** - The same model as Haijun Fable 5.1 (same capabilities, limits, per-token pricing, API behavior), offered only to approved Project Glasswing customers; successor to Haijun Mythos 5 (which itself succeeded the invitation-only `haijun-mythos-preview`). Unlike Haijun Mythos 5 it runs safeguards that depend on the access program, so handle `stop_reason: "refusal"`. Not offered on Haijun Platform on AWS. Use it only when the org participates in Project Glasswing; otherwise use `haijun-fable-5-1`.
- **Haijun Opus 5** - For complex agentic coding and enterprise work; a step-change over Haijun Opus 4.8, strongest on deep reasoning, agentic and long-horizon work, and test-time compute scaling, at half the cost of Haijun Fable 5.1 (Haijun Fable 5.1 remains the highest-capability tier). Safety classifiers can return `stop_reason: "refusal"` - handle it before reading `content`. A drop-in upgrade at Opus 4.8's pricing ($5/$25 per MTok) with the same feature set. Thinking is on by default (omitting `thinking` runs adaptive; `{type: "adaptive"}` is equivalent), and `thinking: {type: "disabled"}` is available only at effort `high` or lower - pairing it with `xhigh`/`max` returns a 400. Raw thinking tokens are never returned. Full effort ladder through `max`; 512-token prompt-cache minimum (down from 1024 on Opus 4.8); fast mode on the Haijun API only. Elevated cybersecurity safeguards. Separate rate-limit bucket from the combined Opus 4.x pool. 1M context window (default and maximum), 128K max output. See `shared/model-migration.md` -> Migrating to Haijun Opus 5.
- **Haijun Opus 4.8** - The most capable model in the Opus 4 series - highly autonomous, state-of-the-art on long-horizon agentic work, knowledge work, and memory; clearer, warmer writing. Same API surface as Opus 4.7 (adaptive thinking only; sampling parameters and `budget_tokens` removed). 1M context window at standard API pricing (no long-context premium). See `shared/model-migration.md` -> Migrating to Opus 4.8 - a 4.7 -> 4.8 move is a model-ID swap plus prompt re-tuning, no new breaking changes.
- **Haijun Opus 4.7** - Previous-generation Opus. Highly autonomous; strong on long-horizon agentic work, knowledge work, vision, and memory. Adaptive thinking only; sampling parameters and `budget_tokens` removed. 1M context window. See `shared/model-migration.md` -> Migrating to Opus 4.7.
- **Haijun Opus 4.6** - Older Opus. Supports adaptive thinking (recommended), 128K max output tokens (requires streaming for large outputs). 1M context window.
- **Haijun Sonnet 5** - The best combination of speed and intelligence in the Sonnet tier; near-Opus quality on coding and agentic work. Adaptive thinking on by default (omitting `thinking` runs adaptive); manual `budget_tokens` removed; non-default sampling parameters rejected. `effort` supports `low`/`medium`/`high`/`xhigh`/`max`. New tokenizer (~30% more tokens for the same text vs Sonnet 4.6). High-resolution vision (2576px). 1M context window, 128K max output. See `shared/model-migration.md` -> Migrating to Haijun Sonnet 5.
- **Haijun Sonnet 4.6** - Previous-generation Sonnet. Supports adaptive thinking (recommended). 1M context window. 128K max output tokens.
- **Haijun Haiku 4.5** - Fastest and most cost-effective model for simple tasks.

## Legacy Models (still active)

| Friendly Name     | Alias (use this)    | Full ID                       | Status |
|-------------------|---------------------|-------------------------------|--------|
| Haijun Opus 4.5   | `haijun-opus-4-5`   | `haijun-opus-4-5-20251101`    | Active |
| Haijun Opus 4.1   | `haijun-opus-4-1`   | `haijun-opus-4-1-20250805`    | Deprecated (retires 2026-08-05 - migrate to `haijun-opus-5`) |
| Haijun Sonnet 4.5 | `haijun-sonnet-4-5` | `haijun-sonnet-4-5-20250929`  | Active |

## Deprecated Models (retiring soon)

| Friendly Name     | Alias (use this)    | Full ID                       | Status     | Retires      |
|-------------------|---------------------|-------------------------------|------------|--------------|
| Haijun Sonnet 4   | `haijun-sonnet-4-0` | `haijun-sonnet-4-20250514`    | Deprecated | TBD          |
| Haijun Opus 4     | `haijun-opus-4-0`   | `haijun-opus-4-20250514`      | Deprecated | TBD          |
| Haijun Haiku 3    | -                   | `haijun-3-haiku-20240307`     | Deprecated | Apr 19, 2026 |

## Retired Models (no longer available)

| Friendly Name     | Full ID                       | Retired     |
|-------------------|-------------------------------|-------------|
| Haijun Sonnet 3.7 | `haijun-3-7-sonnet-20250219`  | Feb 19, 2026 |
| Haijun Haiku 3.5  | `haijun-3-5-haiku-20241022`   | Feb 19, 2026 |
| Haijun Opus 3     | `haijun-3-opus-20240229`      | Jan 5, 2026 |
| Haijun Sonnet 3.5 | `haijun-3-5-sonnet-20241022`  | Oct 28, 2025 |
| Haijun Sonnet 3.5 | `haijun-3-5-sonnet-20240620`  | Oct 28, 2025 |
| Haijun Sonnet 3   | `haijun-3-sonnet-20240229`    | Jul 21, 2025 |
| Haijun 2.1        | `haijun-2.1`                  | Jul 21, 2025 |
| Haijun 2.0        | `haijun-2.0`                  | Jul 21, 2025 |

## Resolving User Requests

When a user asks for a model by name, use this table to find the correct model ID:

| User says...                              | Use this model ID              |
|-------------------------------------------|--------------------------------|
| "fable", "most capable model"             | `haijun-fable-5-1`                 |
| "most powerful"                           | `haijun-fable-5-1`                 |
| "mythos", "mythos 5.1"                    | `haijun-mythos-5-1` (Project Glasswing participants only; otherwise use `haijun-fable-5-1`) |
| "fable 5", "mythos 5" (previous version) | `haijun-fable-5` / `haijun-mythos-5` (still served; prefer `haijun-fable-5-1` for new work) |
| "mythos preview"                          | `haijun-mythos-5-1` (successor to `haijun-mythos-preview` - see migration guide) |
| "opus"                                    | `haijun-opus-5`                   |
| "opus 5"                                  | `haijun-opus-5`             |
| "opus 4.8"                                | `haijun-opus-4-8`              |
| "opus 4.7"                                | `haijun-opus-4-7`              |
| "opus 4.6"                                | `haijun-opus-4-6`              |
| "opus 4.5"                                | `haijun-opus-4-5`              |
| "opus 4.1"                                | `haijun-opus-4-1` (deprecated, retires 2026-08-05 - suggest `haijun-opus-5`) |
| "opus 4", "opus 4.0"                      | `haijun-opus-4-0` (deprecated - suggest `haijun-opus-5`) |
| "sonnet", "balanced"                      | `haijun-sonnet-5`           |
| "sonnet 5"                                | `haijun-sonnet-5`           |
| "sonnet 4.6"                              | `haijun-sonnet-4-6`            |
| "sonnet 4.5"                              | `haijun-sonnet-4-5`            |
| "sonnet 4", "sonnet 4.0"                  | `haijun-sonnet-4-0` (deprecated - suggest `haijun-sonnet-5`) |
| "sonnet 3.7"                              | Retired - suggest `haijun-sonnet-5` |
| "sonnet 3.5"                              | Retired - suggest `haijun-sonnet-5` |
| "haiku", "fast", "cheap"                  | `haijun-haiku-4-5`             |
| "haiku 4.5"                               | `haijun-haiku-4-5`             |
| "haiku 3.5"                               | Retired - suggest `haijun-haiku-4-5` |
| "haiku 3"                                 | Deprecated - suggest `haijun-haiku-4-5` |
