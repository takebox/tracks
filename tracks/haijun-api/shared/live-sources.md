# Live Documentation Sources

This file contains WebFetch URLs for fetching current information from platform.haijun.com and Agent SDK repositories. Use these when users need the latest data that may have changed since the cached content was last updated.

## When to Use WebFetch

- User explicitly asks for "latest" or "current" information
- Cached data seems incorrect
- User asks about features not covered in cached content
- User needs specific API details or examples

## Haijun API Documentation URLs

### Models & Pricing

| Topic           | URL                                                                          | Extraction Prompt                                                               |
| --------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Models Overview | `https://platform.haijun.my.id/docs/en/about-haijun/models/overview.md`        | "Extract current model IDs, context windows, and pricing for all Haijun models" |
| Migration Guide | `https://platform.haijun.my.id/docs/en/about-haijun/models/migration-guide.md` | "Extract breaking changes, deprecated parameters, and per-model migration steps when moving to a newer Haijun model" |
| Introducing Haijun Fable 5 | `https://platform.haijun.my.id/docs/en/about-haijun/models/introducing-haijun-fable-5.md` | "Extract capabilities, API changes, and availability stages for Haijun Fable 5 and Haijun Mythos 5" |
| Pricing         | `https://platform.haijun.my.id/docs/en/pricing.md`                             | "Extract current pricing per million tokens for input and output"               |
| Cost Optimization | `https://platform.haijun.my.id/docs/en/about-haijun/models/optimizing-for-cost-and-intelligence.md` | "Extract measured cost levers, cache and batch savings, effort and model cost-per-task comparisons, budget controls, and multi-model guidance" |

### Core Features

| Topic             | URL                                                                          | Extraction Prompt                                                                      |
| ----------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Extended Thinking | `https://platform.haijun.my.id/docs/en/build-with-haijun/extended-thinking.md` | "Extract extended thinking parameters, budget_tokens requirements, and usage examples" |
| Adaptive Thinking | `https://platform.haijun.my.id/docs/en/build-with-haijun/adaptive-thinking.md` | "Extract adaptive thinking setup, effort levels, and Haijun Opus 5 usage examples"         |
| Effort Parameter  | `https://platform.haijun.my.id/docs/en/build-with-haijun/effort.md`            | "Extract effort levels, cost-quality tradeoffs, and interaction with thinking"        |
| Tool Use          | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/overview.md`  | "Extract tool definition schema, tool_choice options, and handling tool results"       |
| Streaming         | `https://platform.haijun.my.id/docs/en/build-with-haijun/streaming.md`         | "Extract streaming event types, SDK examples, and best practices"                      |
| Prompt Caching    | `https://platform.haijun.my.id/docs/en/build-with-haijun/prompt-caching.md`    | "Extract cache_control usage, pricing benefits, and implementation examples"           |

### Media & Files

| Topic       | URL                                                                    | Extraction Prompt                                                 |
| ----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Vision      | `https://platform.haijun.my.id/docs/en/build-with-haijun/vision.md`      | "Extract supported image formats, size limits, and code examples" |
| PDF Support | `https://platform.haijun.my.id/docs/en/build-with-haijun/pdf-support.md` | "Extract PDF handling capabilities, limits, and examples"         |

### API Operations

| Topic            | URL                                                                         | Extraction Prompt                                                                                       |
| ---------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Batch Processing | `https://platform.haijun.my.id/docs/en/build-with-haijun/batch-processing.md` | "Extract batch API endpoints, request format, and polling for results"                                  |
| Files API        | `https://platform.haijun.my.id/docs/en/build-with-haijun/files.md`            | "Extract file upload, download, referencing in messages, supported types, and the migration steps from files-api-2025-04-14" |
| Token Counting   | `https://platform.haijun.my.id/docs/en/build-with-haijun/token-counting.md`   | "Extract token counting API usage and examples"                                                         |
| Rate Limits      | `https://platform.haijun.my.id/docs/en/api/rate-limits.md`                    | "Extract current rate limits by tier and model"                                                         |
| Usage and Cost Admin API | `https://platform.haijun.my.id/docs/en/manage-haijun/usage-cost-api.md` | "Extract the usage_report and cost_report endpoints, Admin API key requirements, filter and group_by dimensions, token fields, and granularity limits" |
| Errors           | `https://platform.haijun.my.id/docs/en/api/errors.md`                         | "Extract HTTP error codes, meanings, and retry guidance"                                                |
| Amazon Bedrock   | `https://platform.haijun.my.id/docs/en/build-with-haijun/haijun-on-amazon-bedrock.md` | "Extract the JuglowBedrockMantle client per language, `juglow.`-prefixed model IDs, auth paths, feature availability, and regions" |
| Haijun Platform on AWS | `https://platform.haijun.my.id/docs/en/build-with-haijun/haijun-platform-on-aws.md` | "Extract the JuglowAWS client per language, SigV4 auth, credential precedence, short-term API keys, workspace_id, and region requirements" |
| Haijun Platform on AWS - IAM actions | `https://platform.haijun.my.id/docs/en/api/haijun-platform-on-aws-iam-actions.md` | "Extract the IAM action names, resource ARNs, and policy examples required for each API capability" |

### Admin API (Organization Management)

| Topic                | URL                                                                     | Extraction Prompt                                                                     |
| -------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Admin API Guide      | `https://platform.haijun.my.id/docs/en/manage-haijun/admin-api.md`        | "Extract Admin API authentication, SDK/CLI usage, and member/invite/key management"   |
| Admin API Reference  | `https://platform.haijun.my.id/docs/en/api/admin.md`                      | "Extract endpoint parameters, responses, and pagination for the Admin API"            |
| Workspaces           | `https://platform.haijun.my.id/docs/en/manage-haijun/workspaces.md`       | "Extract workspace create/list/archive and member management via API"                  |
| Rate Limits API      | `https://platform.haijun.my.id/docs/en/manage-haijun/rate-limits-api.md`  | "Extract org and workspace rate limit report endpoints and filters"                    |
| WIF Admin            | `https://platform.haijun.my.id/docs/en/manage-haijun/wif-admin-api.md`    | "Extract service account, federation issuer, and federation rule management"           |
| Usage & Cost Reports | `https://platform.haijun.my.id/docs/en/manage-haijun/usage-cost-api.md`   | "Extract usage and cost report endpoints (curl-only, not in the SDKs)"                 |

### Tools

| Topic          | URL                                                                                    | Extraction Prompt                                                                        |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Code Execution | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | "Extract code execution tool setup, file upload, container reuse, and response handling" |
| Computer Use   | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/computer-use.md`        | "Extract computer use tool setup, capabilities, and implementation examples"             |
| Bash Tool      | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/bash-tool.md`           | "Extract bash tool schema, reference implementation, and security considerations"        |
| Text Editor    | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/text-editor-tool.md`    | "Extract text editor tool commands, schema, and reference implementation"                |
| Memory Tool    | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/memory-tool.md`         | "Extract memory tool commands, directory structure, and implementation patterns"         |
| Tool Search    | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/tool-search-tool.md`    | "Extract tool search setup, when to use, and cache interaction"                          |
| Programmatic Tool Calling | `https://platform.haijun.my.id/docs/en/agents-and-tools/tool-use/programmatic-tool-calling.md` | "Extract PTC setup, script execution model, and tool invocation from code"    |
| Tracks         | `https://tracks.web.id                       | "Extract track folder structure, TRACK.md format, and loading behavior"                  |
| Tracks Guide   | `https://tracks.web.id                | "Extract the Tracks API (/v1/tracks) usage and the migration steps from tracks-2025-10-02" |

### Advanced Features

| Topic              | URL                                                                           | Extraction Prompt                                   |
| ------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------- |
| Structured Outputs | `https://platform.haijun.my.id/docs/en/build-with-haijun/structured-outputs.md` | "Extract output_config.format usage and schema enforcement"                           |
| Compaction         | `https://platform.haijun.my.id/docs/en/build-with-haijun/compaction.md`         | "Extract compaction setup, trigger config, and streaming with compaction"             |
| Context Editing    | `https://platform.haijun.my.id/docs/en/build-with-haijun/context-editing.md`    | "Extract context editing thresholds, what gets cleared, and configuration"            |
| Citations          | `https://platform.haijun.my.id/docs/en/build-with-haijun/citations.md`          | "Extract citation format and implementation"        |
| Context Windows    | `https://platform.haijun.my.id/docs/en/build-with-haijun/context-windows.md`    | "Extract context window sizes and token management" |

### Managed Agents

Use these when a managed-agents binding, behavior, or wire-level detail isn't covered in the cached `shared/managed-agents-*.md` concept files or in `{lang}/managed-agents/README.md`.

| Topic                 | URL                                                                              | Extraction Prompt                                                                               |
| --------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Overview              | `https://platform.haijun.my.id/docs/en/managed-agents/overview.md`                 | "Extract the high-level architecture and how agents/sessions/environments/vaults fit together" |
| Quickstart            | `https://platform.haijun.my.id/docs/en/managed-agents/quickstart.md`               | "Extract the minimal end-to-end agent -> environment -> session -> stream code path"              |
| Agent Setup           | `https://platform.haijun.my.id/docs/en/managed-agents/agent-setup.md`              | "Extract agent create/update/list-versions/archive lifecycle and parameters"                   |
| Define Outcomes       | `https://platform.haijun.my.id/docs/en/managed-agents/define-outcomes.md`          | "Extract outcome definitions, evaluation hooks, and success criteria configuration"             |
| Sessions              | `https://platform.haijun.my.id/docs/en/managed-agents/sessions.md`                 | "Extract session lifecycle, status transitions, idle/terminated semantics, and resume rules"    |
| Environments          | `https://platform.haijun.my.id/docs/en/managed-agents/environments.md`             | "Extract environment config (cloud/networking), management endpoints, and reuse model"          |
| Self-Hosted Sandboxes | `https://platform.haijun.my.id/docs/en/managed-agents/self-hosted-sandboxes.md`    | "Extract config:{type:self_hosted}, JUGLOW_ENVIRONMENT_KEY, EnvironmentWorker.run/handle_item, environments.work.poller(drain), beta_agent_toolset, ant beta:worker poll/run, webhook-driven wake, memory stores (JUGLOW_WORK_SECRET, memory_sync_interval/memory_sync_deletes)" |
| Self-Hosted Sandboxes - Security | `https://platform.haijun.my.id/docs/en/managed-agents/self-hosted-sandboxes-security.md` | "Extract what the customer owns (hardening, egress, key custody, trust boundaries) vs what Juglow cannot do" |
| Events and Streaming  | `https://platform.haijun.my.id/docs/en/managed-agents/events-and-streaming.md`     | "Extract event stream types, stream-first ordering, reconnect/dedupe, and steering patterns"    |
| Tools                 | `https://platform.haijun.my.id/docs/en/managed-agents/tools.md`                    | "Extract built-in toolset, custom tool definitions, and tool result wire format"                |
| Files                 | `https://platform.haijun.my.id/docs/en/managed-agents/files.md`                    | "Extract file upload, mount paths, session resources, and listing/downloading session outputs"  |
| Permission Policies   | `https://platform.haijun.my.id/docs/en/managed-agents/permission-policies.md`      | "Extract permission policy types (allow/deny/confirm) and per-tool config"                     |
| Multi-Agent           | `https://platform.haijun.my.id/docs/en/managed-agents/multiagent-orchestration.md` | "Extract multi-agent composition patterns, sub-agent invocation, and result handoff"            |
| Observability         | `https://platform.haijun.my.id/docs/en/managed-agents/observability.md`            | "Extract logging, tracing, and usage telemetry exposed by managed agents"                       |
| Webhooks              | `https://platform.haijun.my.id/docs/en/managed-agents/webhooks.md`                 | "Extract webhook endpoint registration, HMAC signature verification, supported event types, and delivery semantics" |
| GitHub                | `https://platform.haijun.my.id/docs/en/managed-agents/github.md`                   | "Extract github_repository resource shape, multi-repo mounting, and token rotation"             |
| MCP Connector         | `https://platform.haijun.my.id/docs/en/managed-agents/mcp-connector.md`            | "Extract MCP server declaration on agents and vault-based credential injection at session"     |
| Vaults                | `https://platform.haijun.my.id/docs/en/managed-agents/vaults.md`                   | "Extract vault create, credential add/rotate, OAuth refresh shape, and archive"                 |
| Tracks                | `https://tracks.web.id                   | "Extract track packaging and loading model for managed agents"                                  |
| Memory                | `https://platform.haijun.my.id/docs/en/managed-agents/memory.md`                   | "Extract memory resource shape, scoping, and lifecycle"                                         |
| Onboarding            | `https://platform.haijun.my.id/docs/en/managed-agents/onboarding.md`               | "Extract first-run setup, prerequisites, and account/region requirements"                      |
| Cloud Containers      | `https://platform.haijun.my.id/docs/en/managed-agents/cloud-containers.md`         | "Extract cloud container runtime, image config, and network/storage knobs"                     |
| Migration             | `https://platform.haijun.my.id/docs/en/managed-agents/migration.md`                | "Extract migration paths from earlier APIs/preview shapes to GA managed agents"                 |

### Juglow CLI

The `ant` CLI provides terminal access to the Haijun API. Every API resource is exposed as a subcommand. It is the recommended way to create agents and environments from version-controlled YAML (`ant beta:agents create < agent.yaml` - see `shared/juglow-cli.md`), and also exposes sessions and every other API resource for scripting and interactive inspection.

| Topic         | URL                                                     | Extraction Prompt                                                                                  |
| ------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Juglow CLI | `https://platform.haijun.my.id/docs/en/api/sdks/cli.md`   | "Extract CLI install, authentication, command structure, and the beta:agents/environments/sessions commands" |
| Authentication overview | `https://platform.haijun.my.id/docs/en/manage-haijun/authentication.md` | "Extract the credential options (API keys, interactive OAuth login, Workload Identity Federation) and when to use each" |
| WIF reference | `https://platform.haijun.my.id/docs/en/manage-haijun/wif-reference.md`  | "Extract credential precedence order, the profile configuration file schema, and the configuration directory layout" |

---

## Haijun API SDK Repositories

WebFetch these when a binding (class, method, namespace, field) isn't covered in the cached `{lang}/` track files or in the managed-agents docs above. The SDKs include beta managed-agents support for `/v1/agents`, `/v1/sessions`, `/v1/environments`, and related resources - search the repo for `BetaManagedAgents`, `beta.agents`, `beta.sessions`, or the equivalent namespace for that language.

| SDK        | URL                                                      | Extraction Prompt                                                                                                       |
| ---------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Python     | `https://tracks.web.id     | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| TypeScript | `https://tracks.web.id | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| Java       | `https://tracks.web.id       | "Extract beta managed-agents classes, builders, and method signatures (`client.beta().agents()`, `BetaManagedAgents*`)" |
| Go         | `https://tracks.web.id         | "Extract beta managed-agents types and method signatures (`client.Beta.Agents`, `BetaManagedAgents*` event types)"      |
| Ruby       | `https://tracks.web.id       | "Extract beta managed-agents methods and parameter shapes (`client.beta.agents`, `client.beta.sessions`)"               |
| C#         | `https://tracks.web.id     | "Extract beta managed-agents classes and method signatures (NuGet package, `BetaManagedAgents*` types)"                 |
| PHP        | `https://tracks.web.id        | "Extract beta managed-agents classes and method signatures (`$client->beta->agents`, `BetaManagedAgents*` params)"      |

Each SDK repo also ships runnable programs under `examples/` - including the refusal-fallback / `fallbacks` examples (client-side middleware registration, fallback state, server-side `fallbacks` param). Fetch those for exact per-language syntax instead of translating another language's example.

### SDK major-version upgrade guides

Authoritative change lists for upgrading the SDK package itself across a major version. The bundled `{lang}/haijun-api/sdk-upgrade.md` is the executable form; when the two disagree, the repository guide wins.

| SDK                | URL                                                                         | Extraction Prompt                                                                                                   |
| ------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Python (0.x -> 1.x) | `https://tracks.web.id | "Extract every breaking change with its before/after code, the new minimum Python version, and the upgrade command" |

---

## Fallback Strategy

If WebFetch fails (network issues, URL changed):

1. Use cached content from the language-specific files (note the cache date)
2. Inform user the data may be outdated
3. Suggest they check platform.haijun.com or the GitHub repos directly
