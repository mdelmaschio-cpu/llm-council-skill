# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

A Claude skill that enables collaborative brainstorming by querying ChatGPT and Gemini before presenting a synthesized implementation plan. Both APIs are queried in parallel; each model's contributions are attributed in the final response.

## Skill Structure

```
llm-council/
├── SKILL.md                    # Skill instructions and trigger phrases
├── scripts/
│   └── query_llms.py          # Queries OpenAI and Gemini APIs concurrently
├── references/
│   └── SETUP.md               # Detailed setup guide
└── .env.template              # API key configuration template
```

The repo is distributed as `llm-council.skill` (a zip archive of the above). There is no build step.

## Configuration

Users create a `.env` file in their working directory from `.env.template`:

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
# Optional model overrides:
OPENAI_MODEL=gpt-5-nano-2025-08-07     # default (fast, low cost)
GEMINI_MODEL=gemini-3-flash-preview    # default
```

Each invocation makes exactly 2 API calls (one to OpenAI, one to Gemini). The script has a 30-second timeout per API — if one fails, Claude continues with the remaining model's response.

## Activation Phrases

The skill triggers on:
- `Consult the council: <question>`
- `Ask ChatGPT and Gemini <question>`
- `Get perspectives from other AI models on <topic>`
- `Consult with other LLMs: <question>`

## Packaging the .skill File

To regenerate `llm-council.skill` after edits, zip the skill contents from the repo root:

```bash
zip -r llm-council.skill SKILL.md scripts/ references/ .env.template
```

The `.env` file itself is never packaged — users provision it from `.env.template`.

## Model Options

Default models balance cost and speed. Upgrade tiers for higher quality:

| Quality | OpenAI | Gemini |
|---------|--------|--------|
| Budget | `gpt-5-nano` (default) | `gemini-2.5-flash` |
| Standard | `gpt-5-mini` | `gemini-3-flash-preview` (default) |
| Premium | `gpt-5.2-pro` | `gemini-3-pro-preview` |
