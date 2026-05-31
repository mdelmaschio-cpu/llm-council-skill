# CLAUDE.md — llm-council-skill

This file provides guidance to AI assistants working in this repository.

## Project Overview

**LLM Council Skill** is a Claude Code skill that enables Claude to consult ChatGPT and Gemini in parallel before presenting a final synthesized response. When invoked, Claude queries both APIs, analyzes their perspectives, and produces a unified implementation plan with attribution to each model's contributions.

- **No build step** — Markdown skill file + Python query script
- **License**: MIT (see `LICENSE`)

## Repository Layout

```
llm-council-skill/
├── llm-council.skill         # Packaged skill archive (zip)
├── llm-council/
│   ├── SKILL.md              # Main skill instructions (frontmatter + body)
│   ├── scripts/
│   │   └── query_llms.py    # Python script: queries OpenAI + Gemini APIs
│   └── references/
│       └── setup.md          # Setup and configuration reference
├── .env.template             # Template for API key configuration
├── LICENSE
└── README.md
```

## Skill Architecture

### SKILL.md

The skill is invoked by phrases like "consult the council", "ask ChatGPT and Gemini", or "get perspectives from other AI models". It:

1. Calls `scripts/query_llms.py` with the user's question
2. Receives responses from ChatGPT and Gemini (30-second timeout each)
3. Analyzes and compares the perspectives
4. Synthesizes a unified plan with per-model attribution

### query_llms.py

A standalone Python script that:
- Reads API keys from a `.env` file in the current working directory
- Makes concurrent API calls to OpenAI and Gemini
- Returns structured JSON with each model's response
- Handles timeouts and partial failures gracefully (one API down = continue with the other)

### Model Configuration

Default models (fast, cost-effective):
- ChatGPT: `gpt-5-nano-2025-08-07`
- Gemini: `gemini-3-flash-preview`

Configure via `.env`:

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
OPENAI_MODEL=gpt-5-nano      # optional override
GEMINI_MODEL=gemini-3-flash-preview  # optional override
```

## Installation

1. Copy the `llm-council/` directory to your Claude Code skills directory:
   ```bash
   cp -r llm-council ~/.claude/skills/
   ```
   Or install from the `.skill` archive in Claude Code's skill manager.

2. Create a `.env` file in your working directory:
   ```bash
   cp .env.template .env
   # Edit .env with your API keys
   ```

3. Invoke by telling Claude: "Consult the council: \<your question\>"

## API Costs

Each invocation makes 2 API calls (one OpenAI + one Gemini). With defaults (`gpt-5-nano` + `gemini-3-flash-preview`) the cost is very low. Upgrade to premium models (`gpt-5.2-pro`, `gemini-3-pro-preview`) only for critical decisions.

## Development Workflow

### Modifying the Skill Instructions

1. Edit `llm-council/SKILL.md`
2. Update frontmatter `version` if making behavioral changes
3. Test by invoking "consult the council: \<sample question\>" in Claude Code

### Modifying the Query Script

1. Edit `llm-council/scripts/query_llms.py`
2. The script must remain invocable as a standalone CLI: `python3 query_llms.py "<question>"`
3. Output must remain structured JSON for the skill to parse correctly
4. Verify timeout handling still works (both APIs have a 30s per-call timeout)

### Adding a New Model Provider

1. Add the provider's API call in `query_llms.py`
2. Add the corresponding `*_API_KEY` and `*_MODEL` env vars to `.env.template`
3. Update `SKILL.md` to reference the new model in the synthesis step
4. Update `README.md` model options table

## Key Conventions

- **`.env` in working directory** — API keys must never be committed; the `.env.template` is committed; the `.env` itself is gitignored
- **Graceful degradation** — if one API fails, the skill continues with the available response and notes the outage; do not error out completely
- **Attribution required** — the synthesized output must always attribute which ideas came from which model
- **30-second timeout** — do not increase the per-API timeout; if an API is consistently slow, note it in the output rather than waiting indefinitely
- **Structured JSON output from script** — `query_llms.py` must always return valid JSON; parse errors in the skill should be surfaced clearly to the user
