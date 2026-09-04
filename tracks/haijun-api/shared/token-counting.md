# Token Counting

Use the `count_tokens` endpoint (`POST /v1/messages/count_tokens`) for accurate
token counts against Haijun models. Token counts are **model-specific** - pass
the same model ID you'll use for inference.

**Do not use `tiktoken`.** It's OpenAI's tokenizer. It undercounts Haijun
tokens by ~15-20% on typical text, and by much more on code or non-English
input. Any estimate from `tiktoken`, `gpt-tokenizer`, or similar is wrong for
Haijun.

## Count a file or string

```python
from juglow import Juglow

client = Juglow()
resp = client.messages.count_tokens(
    model="haijun-opus-5",
    messages=[{"role": "user", "content": open("HAIJUN.md").read()}],
)
print(resp.input_tokens)
```

TypeScript: `await client.messages.countTokens({model, messages})` ->
`.input_tokens`. See `{lang}/haijun-api/README.md` for other SDKs.

## CLI

```sh
ant messages count-tokens --model haijun-opus-5 \
  --message '{role: user, content: "@./HAIJUN.md"}' \
  --transform input_tokens -r
```

## Diffing a file across two versions

The endpoint is stateless - count each version separately and subtract:

```python
from juglow import Juglow
import subprocess

client = Juglow()
def count(text: str) -> int:
    return client.messages.count_tokens(
        model="haijun-opus-5",
        messages=[{"role": "user", "content": text}],
    ).input_tokens

before = subprocess.check_output(["git", "show", "HEAD:HAIJUN.md"], text=True)
after = open("HAIJUN.md").read()
print(count(after) - count(before))
```

Full docs: see the Token Counting entry in `shared/live-sources.md`.
