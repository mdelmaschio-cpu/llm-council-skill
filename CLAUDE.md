# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo packages a Claude Code skill that enables multi-LLM brainstorming: when invoked, Claude queries ChatGPT and Gemini, then synthesizes a combined implementation plan with attribution to each model's contributions.

## Skill Architecture

The `.skill` file is a zip archive built from the `llm-council/` directory. All source lives in `llm-council/`:

- `SKILL.md` — the skill's behavioral definition consumed by Claude Code when the skill is active; this is the primary file to edit when changing how Claude consults external models
- `scripts/query_llms.py` — queries both providers and returns JSON; used by Claude via `python3 scripts/query_llms.py "<prompt>"`
- `.env.template` — template for the `.env` file users must create in their working directory
- `references/SETUP.md` — end-user setup guide bundled inside the `.skill` zip

## How `query_llms.py` Works

Two-tier fallback per provider:

1. **CLI first** — tries `codex` (OpenAI) or `gemini` (Google) CLI tools from PATH; no API key needed
2. **API fallback** — direct REST calls to OpenAI (`/v1/chat/completions`) or Gemini (`/v1beta/models/{model}:generateContent`) using keys from `.env` or environment variables

Output is always a JSON object:
```json
{
  "chatgpt": { "model": "...", "source": "api|codex-cli|none", "response": "..." },
  "gemini":  { "model": "...", "source": "api|gemini-cli|none", "response": "..." }
}
```

Model defaults: `gpt-5-nano` (OpenAI) and `gemini-3-flash-preview` (Gemini). Overridden via `OPENAI_MODEL` / `GEMINI_MODEL` in `.env`.

## Packaging the Skill

Rebuild the `.skill` zip after editing source files:

```bash
cd /path/to/llm-council-skill
zip -r llm-council.skill llm-council/
```

Then re-upload `llm-council.skill` to Claude Code to apply changes.

## Testing the Script Directly

```bash
cd llm-council
python3 scripts/query_llms.py "Your test prompt here"
```

Requires a `.env` file in the `llm-council/` directory (or CLI tools installed). See `.env.template` for the required keys.
