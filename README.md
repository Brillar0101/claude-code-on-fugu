# Claude Code on Sakana Fugu

Run the Claude Code CLI against [Sakana Fugu](https://console.sakana.ai/get-started) instead of Anthropic's models.

Claude Code talks to the Anthropic API. Fugu exposes an OpenAI-style API. They don't speak the same wire format, so you run [claude-code-router](https://github.com/musistudio/claude-code-router) locally and it translates between the two.

Heads up: this isn't an official integration. Sakana documents Codex, not Claude Code. The router approach works fine in practice but it's community tooling, so don't be shocked if a tool call occasionally acts up.

## What you need

- Node 18 or newer (npm comes with it)
- A Fugu API key from https://console.sakana.ai (you only see it once, so copy it right away)

## Install

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @musistudio/claude-code-router
```

## Config

Put this in `~/.claude-code-router/config.json`:

```json
{
  "LOG": true,
  "API_TIMEOUT_MS": 600000,
  "Providers": [
    {
      "name": "sakana",
      "api_base_url": "https://api.sakana.ai/v1/responses",
      "api_key": "YOUR_SAKANA_API_KEY",
      "models": ["fugu", "fugu-ultra"],
      "transformer": {
        "use": ["openai-responses"]
      }
    }
  ],
  "Router": {
    "default": "sakana,fugu-ultra",
    "background": "sakana,fugu",
    "think": "sakana,fugu-ultra",
    "longContext": "sakana,fugu-ultra"
  }
}
```

Swap in your own key for `YOUR_SAKANA_API_KEY`.

A couple of things that matter here:

- The base URL points at `/v1/responses` (Fugu's Responses API), not `/v1/chat/completions`. If you use chat completions you'll hit `Invalid 'max_tokens': must be greater than or equal to 16` because Claude Code fires off some tiny background requests. The Responses path drops `max_tokens` so the problem goes away.
- `openai-responses` is the transformer that handles that endpoint and the Anthropic conversion.
- The 10 minute timeout is there because Fugu is a multi-agent system and some turns take a while.

## Run it

```bash
ccr restart
ccr status   # should say Running on port 3456
ccr code
```

`ccr code` opens Claude Code wired to the router. Pick the model inside the session:

```
/model sakana,fugu-ultra
```

Then throw something at it, e.g. ask it to create a file, and check your usage on the Sakana console to confirm the call actually landed on Fugu.

## Models

- `fugu` - default, routes across the supported providers
- `fugu-ultra` - the bigger one

Both take a reasoning effort of `high` or `xhigh` (`max` maps to `xhigh`).

## Commands

- `ccr code` - start Claude Code on Fugu
- `ccr status` - is the router up
- `ccr restart` - run this after any config edit
- `ccr stop` - shut it down

## If something breaks

- `max_tokens` 400 error: double check the base URL ends in `/v1/responses` and the transformer is `openai-responses`, then `ccr restart`.
- 401: wrong key. Fix the config and restart.
- Tool calls or edits going sideways: try `"use": ["openai-responses", "tooluse"]` and restart.
- Not sure it's hitting Fugu: watch the usage at https://console.sakana.ai.

## Don't commit your key

The real config with your key lives at `~/.claude-code-router/config.json`, outside this repo. Keep it that way. If a key ever leaks, rotate it from the console.

---

Not affiliated with Anthropic or Sakana. Names belong to their owners.
