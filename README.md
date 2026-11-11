# Open Uncensored agent skill

An [agent skill](https://skills.sh) that teaches Claude Code, Codex, Cursor, OpenCode and other coding agents to use
[Open Uncensored](https://openuncensored.com): one OpenAI compatible API for uncensored and abliterated models
(DeepSeek, GLM, Qwen, Gemma, MiMo and more), ranked by measured tests, with no account: a prepaid key, provider prices plus a 20% platform fee.

## Install

```bash
npx skills add DhruvAtreja/open-uncensored-skills
```

## What the agent learns

- Pick a model from the live rankings (agent reliability, refusals, price) instead of guessing.
- Get an API key without a browser: start a checkout, hand the payment link to you, collect the key.
- Call chat, Responses and image endpoints, with routing and fallbacks.
- Set Open Uncensored up as a provider in coding tools, or add the MCP server.

The skill is in [`skills/open-uncensored/SKILL.md`](skills/open-uncensored/SKILL.md). The full, always current guide
for agents is https://openuncensored.com/agents.md.

## License

MIT
