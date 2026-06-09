# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A Claude skill that enables multi-LLM collaborative brainstorming. When a user triggers the skill (e.g. "Consult the council: ..."), Claude runs `scripts/query_llms.py` to collect perspectives from ChatGPT and Gemini, then synthesizes a plan with attribution to each model's contributions.

The distributable artifact is `llm-council.skill` — a ZIP archive containing `SKILL.md`, `scripts/`, and `references/`.

## Commands

Test the core script directly:
```bash
python3 llm-council/scripts/query_llms.py "Your test prompt here"
```

The script requires either CLI tools (`gemini`, `codex`) in PATH or a `.env` file with API keys. Copy the template to create one:
```bash
cp llm-council/.env.template .env
```

Repackage the `.skill` file after changes:
```bash
cd llm-council && zip -r ../llm-council.skill .
```

## Architecture

**Skill invocation flow:**
1. User triggers with phrases like "Consult the council:", "Ask ChatGPT and Gemini", "Get perspectives from other AI models"
2. Claude extracts the technical question and runs `python3 scripts/query_llms.py "<question>"`
3. Script outputs JSON with `chatgpt` and `gemini` keys, each containing `model`, `source`, and `response`
4. Claude synthesizes responses into a plan with inline attribution

**`query_llms.py` — priority-based querying:**
- For each model, tries a CLI tool first (`codex` for ChatGPT, `gemini` for Gemini)
- Falls back to direct HTTP API calls if the CLI is unavailable
- `source` field in the JSON output reflects which path was taken (`codex-cli`, `api (gpt-5-nano)`, etc.)
- Both APIs use temperature 0.7 and max 2000 output tokens; 30s timeout per HTTP call
- Env vars are loaded from `.env` in the current working directory, with `os.environ` as fallback

**Key files:**
- `llm-council/SKILL.md` — the skill manifest (name, description, workflow, output format, error handling); this is what Claude reads at runtime
- `llm-council/scripts/query_llms.py` — the only executable; no external dependencies beyond `requests`
- `llm-council/.env.template` — documents all supported env vars

## Environment Variables

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `OPENAI_API_KEY` | If no `codex` CLI | — | OpenAI API auth |
| `GEMINI_API_KEY` | If no `gemini` CLI | — | Gemini API auth |
| `OPENAI_MODEL` | No | `gpt-5-nano` | Which OpenAI model to call |
| `GEMINI_MODEL` | No | `gemini-3-flash-preview` | Which Gemini model to call |

The `.env` file must live in the **working directory where Claude is running**, not necessarily the repo root.

## Modifying the Skill Behavior

The skill's trigger phrases, workflow steps, output format, and error handling are all defined declaratively in `llm-council/SKILL.md`. Changes to how Claude responds to council requests go there. Changes to how the APIs are queried (models, parameters, timeout, fallback logic) go in `query_llms.py`.

After any change to files inside `llm-council/`, rebuild the `.skill` ZIP before distributing.
