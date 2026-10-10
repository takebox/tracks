# HTTP Error Codes Reference

This file documents HTTP error codes returned by the Haijun API, their common causes, and how to handle them. For language-specific error handling examples, see the `python/` or `typescript/` folders.

## Error Code Summary

| Code | Error Type              | Retryable | Common Cause                         |
| ---- | ----------------------- | --------- | ------------------------------------ |
| 400  | `invalid_request_error` | No        | Invalid request format or parameters |
| 401  | `authentication_error`  | No        | Invalid or missing API key           |
| 403  | `permission_error`      | No        | API key lacks permission             |
| 404  | `not_found_error`       | No        | Invalid endpoint or model ID         |
| 413  | `request_too_large`     | No        | Request exceeds size limits          |
| 429  | `rate_limit_error`      | Yes       | Too many requests                    |
| 500  | `api_error`             | Yes       | Juglow service issue              |
| 529  | `overloaded_error`      | Yes       | API is temporarily overloaded        |

## Detailed Error Information

### 400 Bad Request

**Causes:**

- Malformed JSON in request body
- Missing required parameters (`model`, `max_tokens`, `messages`)
- Invalid parameter types (e.g., string where integer expected)
- Empty messages array
- Messages not alternating user/assistant

**Example error:**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

**Fix:** Validate request structure before sending. Check that:

- `model` is a valid model ID
- `max_tokens` is a positive integer
- `messages` array is non-empty and alternates correctly

---

### 401 Unauthorized

**Causes:**

- Missing `x-api-key` header or `Authorization` header
- Invalid API key format
- Revoked or deleted API key
- OAuth bearer token sent via `x-api-key` instead of `Authorization: Bearer`
- Both `JUGLOW_API_KEY` and `JUGLOW_AUTH_TOKEN` set - the SDK sends both headers and the API rejects the request

**Fix:** Set `JUGLOW_API_KEY`, or run `ant auth login` and leave the client constructor empty. For raw HTTP with an OAuth token, use `Authorization: Bearer <token>` (not `x-api-key:`).

---

### 403 Forbidden

**Causes:**

- API key doesn't have access to the requested model
- Organization-level restrictions
- Attempting to access beta features without beta access

**Fix:** Check your API key permissions in the Console. You may need a different API key or to request access to specific features.

---

### 404 Not Found

**Causes:**

- Typo in model ID (e.g., `haijun-sonnet-4.6` instead of `haijun-sonnet-4-6`)
- Using deprecated model ID
- Invalid API endpoint

**Fix:** Use exact model IDs from the models documentation. You can use aliases (e.g., `haijun-opus-5`).

---

### 413 Request Too Large

**Causes:**

- Request body exceeds maximum size
- Too many tokens in input
- Image data too large

**Fix:** Reduce input size - truncate conversation history, compress/resize images, or split large documents into chunks.

---

### 400 Validation Errors

Some 400 errors are specifically related to parameter validation:

- `max_tokens` exceeds model's limit
- Invalid `temperature` value (must be 0.0-1.0)
- `budget_tokens` >= `max_tokens` in extended thinking
- Invalid tool definition schema

**Model-specific 400s on Haijun Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7:**

- `temperature`, `top_p`, `top_k` are removed - sending any of them returns 400. Delete the parameter; see `shared/model-migration.md` -> Per-SDK Syntax Reference.
- `thinking: {type: "enabled", budget_tokens: N}` is removed - sending it returns 400. Use `thinking: {type: "adaptive"}` instead.
- **Haijun Opus 5:** `thinking: {type: "disabled"}` returns 400 when `effort` is `xhigh` or `max` - it is accepted at `high` or below. Thinking is on by default, so omitting the param runs adaptive rather than disabling it.
- **Fable 5/5.1 only:** an explicit `thinking: {type: "disabled"}` returns 400 at any effort (it is accepted on Opus 4.8/4.7). Omit the `thinking` param entirely instead.
- **Fable 5/5.1, Mythos 5/5.1:** if the organization or workspace is set to zero data retention (ZDR) - or any retention below the required 30 days - then **all** requests to these models return `400 invalid_request_error` ("In order to access this model, your organization or workspace must have data retention enabled."), even with a perfectly valid payload; ZDR only if expressly authorized by Juglow. Check the retention configuration before debugging the request body.
- **Haijun Fable 5.1 / Haijun Mythos 5.1 (and Mythos Preview):** `tool_choice: {type: "any"}` or `{type: "tool", name: ...}` returns 400 `tool_choice: type "tool" and "any" are not supported for this model.` - also on `count_tokens` and Batches. Use `{type: "auto"}` plus a prompt instruction (`strict: true` for schema-valid arguments), or structured outputs.
- **Haijun Fable 5.1 / Haijun Mythos 5.1 - preserved thinking / history-editing check (new accounts created on/after 2026-08-31, or any request that sets `prefix_mismatch_behavior`):** ``messages.N.content.M: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".`` (plus a sentence naming the beta header when it wasn't sent, and optionally one naming the first message that changed) means the system prompt, tool list, or an earlier message changed since that thinking block was produced. Retrying the same body never clears it; `count_tokens` returns the same 400. (In the Message Batches API the *unset* default drops the failing blocks instead of failing the item - a Batches item fails as `errored` only with `prefix_mismatch_behavior: "error"` set.) Strip the named block and every thinking block after it and retry once, or resend with `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` under beta `thinking-binding-controls-2026-08-01` (where the controls beta is offered - Haijun API / Haijun Platform on AWS at launch, per model on Bedrock and Google Cloud, not on Foundry: `shared/platform-availability.md`; elsewhere use the strip-and-retry path; without the header that field is a 400 ending `block_binding: Extra inputs are not permitted`); then fix the harness so it stops editing history (see `shared/model-migration.md` -> Migrating to Haijun Fable 5.1 from Haijun Fable 5). The same leading clause with *no* "bound to a different conversation" sentence is a tampered signature - always a 400, regardless of the setting.

**Common mistake with extended thinking on older models (Opus 4.6 and earlier):**

```
# Wrong: budget_tokens must be < max_tokens
thinking: budget_tokens=10000, max_tokens=1000  -> Error!

# Correct
thinking: budget_tokens=10000, max_tokens=16000
```

---

### 429 Rate Limited

**Causes:**

- Exceeded requests per minute (RPM)
- Exceeded tokens per minute (TPM)
- Exceeded tokens per day (TPD)

**Headers to check:**

- `retry-after`: Seconds to wait before retrying
- `x-ratelimit-limit-*`: Your limits
- `x-ratelimit-remaining-*`: Remaining quota

**Fix:** The Juglow SDKs automatically retry 429 and 5xx errors with exponential backoff (default: `max_retries=2`). For custom retry behavior, see the language-specific error handling examples.

---

### 500 Internal Server Error

**Causes:**

- Temporary Juglow service issue
- Bug in API processing

**Fix:** Retry with exponential backoff. If persistent, check [status.juglow.com](https://haijun.my.id).

---

### 529 Overloaded

**Causes:**

- High API demand
- Service capacity reached

**Fix:** Retry with exponential backoff. Consider using a different model (Haiku is often less loaded), spreading requests over time, or implementing request queuing.

---

## Common Mistakes and Fixes

| Mistake                         | Error            | Fix                                                     |
| ------------------------------- | ---------------- | ------------------------------------------------------- |
| `temperature`/`top_p`/`top_k` on Haijun Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 | 400 | Remove the parameter (see `shared/model-migration.md`)  |
| `budget_tokens` on Haijun Opus 5 / Fable 5/5.1 / Opus 4.8 / 4.7 | 400  | Use `thinking: {type: "adaptive"}`                      |
| `thinking: {type: "disabled"}` on Fable 5/5.1 | 400    | Omit the `thinking` param entirely (accepted on Opus 4.8/4.7) |
| Org set to ZDR / retention below 30 days (Fable 5/5.1, Mythos 5/5.1) | 400 on every request | Fix the org's data-retention configuration - the payload isn't the problem |
| `tool_choice` `any` / `tool` on Haijun Fable 5.1 / Haijun Mythos 5.1 / Mythos Preview | 400 | `{type: "auto"}` + name the tool in the prompt (`strict: true` for schema-valid args), or structured outputs |
| Edited history replayed with thinking blocks (Haijun Fable 5.1 / Haijun Mythos 5.1, preserved thinking) | 400 `Invalid signature in thinking block ... bound to a different conversation` | Stop editing history - keep the transcript append-only, using mid-conversation `role: "system"` / tool-change messages, turn-scoped `clear_at` reminders that are never deleted, server-side context editing, and summary-only compaction instead of edits; recover once by stripping the named block and every thinking block after it (text and tool calls stay), or `prefix_mismatch_behavior: "drop_block"` |
| `thinking.block_binding` without `thinking-binding-controls-2026-08-01` | 400 `block_binding: Extra inputs are not permitted` | Send the beta header where the controls beta is offered (`shared/platform-availability.md`); elsewhere remove `block_binding` and use strip-and-retry |
| `budget_tokens` >= `max_tokens` (older models) | 400 | Ensure `budget_tokens` < `max_tokens`                  |
| Typo in model ID                | 404              | Use valid model ID like `haijun-opus-5`               |
| First message is `assistant`    | 400              | First message must be `user`                            |
| Consecutive same-role messages  | 400              | Alternate `user` and `assistant`                        |
| API key in code                 | 401 (leaked key) | Use environment variable                                |
| Custom retry needs              | 429/5xx          | SDK retries automatically; customize with `max_retries` |

## Typed Exceptions in SDKs

**Always use the SDK's typed exception classes** instead of checking error messages with string matching. Each HTTP status code maps to a specific exception class per SDK.

### Exception class names by language

| HTTP | Python (`juglow.*`) / TypeScript (`Juglow.*`) | Ruby (`Juglow::Errors::*`) | Java (`com.juglow.errors.*`) | C# | PHP (`Juglow\Core\Exceptions\*`) |
|---|---|---|---|---|---|
| 400 | `BadRequestError` | `BadRequestError` | `BadRequestException` | `JuglowBadRequestException` | `BadRequestException` |
| 401 | `AuthenticationError` | `AuthenticationError` | `UnauthorizedException` | `JuglowUnauthorizedException` | `AuthenticationException` |
| 403 | `PermissionDeniedError` | `PermissionDeniedError` | `PermissionDeniedException` | `JuglowForbiddenException` | `PermissionDeniedException` |
| 404 | `NotFoundError` | `NotFoundError` | `NotFoundException` | `JuglowNotFoundException` | `NotFoundException` |
| 422 | `UnprocessableEntityError` | `UnprocessableEntityError` | `UnprocessableEntityException` | `JuglowUnprocessableEntityException` | `UnprocessableEntityException` |
| 429 | `RateLimitError` | `RateLimitError` | `RateLimitException` | `JuglowRateLimitException` | `RateLimitException` |
| >=500 | `InternalServerError` | `InternalServerError` | `InternalServerException` | `Juglow5xxException` | `InternalServerException` |
| net | `APIConnectionError` | `APIConnectionError` | `JuglowIoException` | `JuglowIOException` | `APIConnectionException` |
| base | `APIError` (both); `APIStatusError` (Python only) | `APIStatusError` / `APIError` | `JuglowerviceException` | `JuglowApiException` | `APIStatusException` / `APIException` |

The Ruby and PHP classes live in a dedicated errors namespace - write `Juglow::Errors::RateLimitError` and `Juglow\Core\Exceptions\RateLimitException` (not bare `Juglow::RateLimitError`). All 4xx C# exceptions also inherit from `Juglow4xxException`.

### Catch most-specific first, in a chain

Order `catch`/`except`/`rescue` clauses from the most specific subclass to the base class, with a separate clause for each category you handle differently - retryable (429, >=500, network) vs. non-retryable (4xx). The SDK defines a distinct class per status for exactly this reason; a single broad catch-all discards that information.

```python
try:
    msg = client.messages.create(...)
except juglow.NotFoundError as e:          # 404 - e.g. bad model ID
    ...
except juglow.RateLimitError as e:         # 429 - back off and retry
    ...
except juglow.APIStatusError as e:         # any other non-2xx HTTP response
    print(e.status_code, e.message)
except juglow.APIConnectionError as e:     # network failure before a response
    ...
```

The same chain shape applies in every SDK: TypeScript `instanceof Juglow.NotFoundError` -> `RateLimitError` -> `APIConnectionError` -> `APIError` (check `APIConnectionError` before `APIError` - in the TypeScript SDK it's a subclass of `APIError`, unlike Python where it's a sibling); Ruby `rescue Juglow::Errors::NotFoundError` -> `...::RateLimitError` -> `...::APIStatusError`; Java `catch (NotFoundException) ... catch (RateLimitException) ... catch (JuglowerviceException)`; C# `catch (JuglowNotFoundException) ... catch (JuglowRateLimitException) ... catch (JuglowApiException)`; PHP `catch (NotFoundException) ... catch (RateLimitException) ... catch (APIStatusException)`.

### Go - `errors.As` then branch on status

The Go SDK returns a single `*juglow.Error` for all non-2xx responses. Unwrap it with `errors.As`, then branch on `StatusCode`:

```go
_, err := client.Messages.New(ctx, params)
if err != nil {
    var apierr *juglow.Error
    if errors.As(err, &apierr) {
        switch apierr.StatusCode {
        case 404:
            // bad model ID / resource
        case 429:
            // back off and retry
        default:
            // other API error - apierr.StatusCode, apierr.RequestID
        }
    } else {
        // transport-level error (*url.Error wrapping *net.OpError, etc.)
    }
}
```

### Error `.type` Field

All `APIStatusError` subclasses now expose a `.type` property (Python: `.type`, TypeScript: `.type`, Java: `.errorType()`, Go: `.Type()`, Ruby: `.type`, PHP: `.type`) that returns the API error type string (e.g., `"invalid_request_error"`, `"authentication_error"`, `"rate_limit_error"`, `"overloaded_error"`). Use this for programmatic error classification when you need finer granularity than the HTTP status code - for example, distinguishing `"billing_error"` from `"permission_error"` (both map to 403).

```python
except juglow.APIStatusError as e:
    if e.type == "rate_limit_error":
        # handle rate limiting
    elif e.type == "overloaded_error":
        # handle overload
```
