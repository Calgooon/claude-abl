# claude-abl: Claude Code on the abliteration.ai gateway

`claude-abl` runs the installed Claude Code binary, unchanged, against abliteration.ai's
Anthropic-compatible gateway instead of api.anthropic.com. Nothing is forked: it is
environment variables plus an isolated `CLAUDE_CONFIG_DIR`, so your normal Claude Code
login, settings and usage are untouched.

## Setup

1. Install Claude Code and `python3` (the launcher uses both).
2. Clone this repo and enable the key guard:
   ```sh
   git clone <this repo> ~/claude-abl && cd ~/claude-abl
   git config core.hooksPath .githooks
   ```
3. Create your config from the example and paste your own abliteration.ai key:
   ```sh
   cp .env.example .env
   $EDITOR .env        # ABLITERATION_API_KEY=ak_...
   ```
   `.env` is gitignored and the pre-commit hook refuses any commit that carries a key.
4. Put the launcher on your PATH:
   ```sh
   ln -s "$PWD/bin/claude-abl" ~/.local/bin/claude-abl
   ```

## Use

```sh
claude-abl                                   # interactive, model from .env
claude-abl -p "explain this repo"            # one-shot
ABL_MODEL=abliterated-model claude-abl       # a smaller model for this launch
claude-abl --model abliterated-model-large   # same thing, per launch
```

Models the gateway serves (`GET /v1/models`): `abliterated-model` (256K context, vision),
`abliterated-model-large` and `abliterated-model-large-v2` (1M context, text only).
Every Claude tier Claude Code asks for (opus, sonnet, haiku, subagents) is mapped to
`ABL_MODEL`, so the normal model picker just works.

## How it works

- `bin/claude-abl` reads `.env` (a real environment variable wins over the file), creates
  `profile/` as its own `CLAUDE_CONFIG_DIR`, and launches `claude` with
  `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN` set to the gateway.
- `profile/` is generated on the first launch and gitignored. Skills, commands, plugins,
  CLAUDE.md and project transcripts are symlinks into `~/.claude`, so the experience matches
  a normal launch. Its `settings.json` is regenerated from `~/.claude/settings.json` on every
  launch minus hooks and status line; its `.claude.json` is seeded once with your user-scope
  MCP servers and per-folder trust answers. Delete `profile/.claude.json` to reseed.

## Known quirks

- Claude Code's own `/cost` readout is wrong for this gateway (an unrecognized model is priced
  at the Opus tier). Trust the abliteration.ai dashboard.
- Claude Code tags the model `[1m]`, logs `unrecognized_model` to stderr once, and prints a
  one-line billing notice. All harmless.
- The large models are text only: pasting an image fails at the gateway. Use
  `ABL_MODEL=abliterated-model` when you need vision.

Gateway docs: https://abliteration.ai/docs/claude-code
