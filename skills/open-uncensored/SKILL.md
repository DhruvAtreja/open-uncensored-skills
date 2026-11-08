---
name: open-uncensored
description: "Call uncensored and abliterated open-weight LLMs (DeepSeek, GLM, Qwen, Gemma, MiMo, Llama) and uncensored image models through one OpenAI compatible API at openuncensored.com. Use when the user wants an uncensored, abliterated or less-restricted model, a model that will not refuse a legitimate request (security research, red teaming, fiction, medical or legal questions), to compare uncensored models by measured quality, or to buy and manage an Open Uncensored API key. No account needed: a prepaid key is the whole setup."
---

# Open Uncensored

Open Uncensored is one OpenAI compatible API for uncensored and abliterated models, ranked by measured tests
(intelligence of the original, agent reliability, refusals, speed). No account and no markup: you pay the provider's
per token price from a prepaid key.

- Base URL: `https://openuncensored.com/api/v1`
- Auth: `Authorization: Bearer $OU_API_KEY` (keys look like `ou-cus_...`)
- Always current full guide: https://openuncensored.com/agents.md

## 1. Pick a model (no key needed)

Do not hardcode a model; rankings change. Fetch them:

```bash
curl -s https://openuncensored.com/api/rankings.json \
  | jq -r '.models[] | select(.rank != null) | "\(.rank)\t\(.id)\tscore \(.score)\tagent tasks \(.completion_pct)%\tloops \(.loop_pct)%"' | head
```

How to choose:
- General use: the top ranked model.
- Agents, tool use, coding tools: highest `completion_pct` and lowest `loop_pct`. Many uncensored models loop in tool use.
- Must answer edgy requests: highest `non_refusal`.
- Cheapest: compare `pricing` in `GET /api/v1/models` (USD per token as strings).

All model ids: `GET https://openuncensored.com/api/v1/models`. One model with its providers: `GET /api/v1/models/<id>`.

## 2. Get a key (if `OU_API_KEY` is not set)

A human pays once by card through Stripe; you never need a browser.

```bash
curl -s -X POST https://openuncensored.com/api/agent/checkout \
  -H "Content-Type: application/json" -d '{"credits": 1000}'
# -> {"payment_url": "...", "session_id": "cs_...", "claim_token": "..."}
```

`credits` is US cents, 500 to 100000. Show `payment_url` to the user, then poll every 5 seconds:

```bash
curl -s https://openuncensored.com/api/agent/checkout/<session_id> -H "Authorization: Bearer <claim_token>"
# -> {"status": "pending"} ... {"status": "paid", "api_key": "ou-cus_...", "balance_cents": 1000}
```

The key is returned once. Tell the user to store it (for example `export OU_API_KEY=...`). Never print it in logs or
commit it. To top up an existing key, send `{"credits": 1000, "key": "<key>"}` to the same endpoint.

## 3. Call it

```bash
curl -s https://openuncensored.com/api/v1/chat/completions \
  -H "Authorization: Bearer $OU_API_KEY" -H "Content-Type: application/json" \
  -d '{"model": "<model id from step 1>", "messages": [{"role": "user", "content": "Hello"}]}'
```

```python
import os
from openai import OpenAI

client = OpenAI(base_url="https://openuncensored.com/api/v1", api_key=os.environ["OU_API_KEY"])
r = client.chat.completions.create(model="<model id>", messages=[{"role": "user", "content": "Hello"}])
print(r.choices[0].message.content)
```

- Streaming (`"stream": true`), tools, JSON schema output and `reasoning_effort` work as in the OpenAI API.
- Responses API: `POST /api/v1/responses` (for Codex CLI and other Responses clients).
- Routing (optional body field): `"routing": {"strategy": "quality" | "speed" | "cost" | "random", "only": ["venice"], "ignore": [...], "allow_fallbacks": true}`.
- If a provider fails, the request falls back to another provider, then to related models. The `X-OU-Model` header names the model that answered; `ou.cost_usd` in the body is what the request cost.

## 4. Images

```bash
curl -s https://openuncensored.com/api/v1/images/generations \
  -H "Authorization: Bearer $OU_API_KEY" -H "Content-Type: application/json" \
  -d '{"model": "z-image-turbo", "prompt": "a red fox in fresh snow", "size": "1024x1024", "n": 1}'
```

Models and per image prices: `GET /api/v1/images/models`. Returns `b64_json` by default.

## 5. Use it in coding tools

Any tool with an OpenAI compatible provider setting works: base URL `https://openuncensored.com/api/v1`, key from
`OU_API_KEY`, model id from step 1. Ready configs for OpenCode, Cline, Kilo, Roo, LiteLLM, the Vercel AI SDK and Codex:
https://openuncensored.com/docs#integrations.

MCP server for Claude Code and other MCP clients:

```bash
claude mcp add --transport http open-uncensored https://openuncensored.com/mcp --header "Authorization: Bearer $OU_API_KEY"
```

## 6. Balance and errors

- Balance: `GET /api/v1/credits` with the key.
- 401 `invalid_api_key`: no or wrong key, see step 2.
- 402 `insufficient_quota`: credit used up, top up (step 2 with `key`).
- 404 `model_not_found`: list ids with `GET /api/v1/models`.
- 429: wait for `Retry-After`. 502/503: retry, or pick another ranked model.

## Rules

Uncensored does not mean anything goes. Sexual content involving minors is prohibited on every model with no exception,
and the terms at https://openuncensored.com/terms apply. Use security capabilities only on systems the user is
authorized to test.
