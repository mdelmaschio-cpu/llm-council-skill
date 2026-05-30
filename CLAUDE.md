# llm-council-skill

A Claude Code skill that enables multi-LLM collaborative brainstorming — Claude queries ChatGPT and Gemini, synthesizes all three perspectives, and presents a unified implementation plan.

## Repository Purpose

Provides the `llm-council` skill. When a user asks Claude to "consult the council" or "ask other AI models," Claude runs a Python script that queries both OpenAI and Gemini APIs in parallel, then synthesizes the results with its own analysis into a single attributed plan.

## Structure

```
llm-council-skill/
├── README.md
├── LICENSE
├── llm-council.skill            # Installable skill bundle (ZIP of llm-council/)
└── llm-council/
    ├── SKILL.md                 # Authoritative skill definition (frontmatter + body)
    ├── .env.template            # Copy to .env and add API keys; never commit .env
    ├── scripts/
    │   └── query_llms.py        # Python script: queries OpenAI + Gemini APIs in parallel
    └── references/
        └── setup.md             # Detailed setup walkthrough for users
```

## How the Skill Works

**Trigger phrases:** "Consult the council", "ask ChatGPT and Gemini", "get perspectives from other AIs", "ask other AI models"

**Execution flow:**
1. Claude calls `python3 llm-council/scripts/query_llms.py "<user prompt>"`
2. Script reads API keys and optional model overrides from `.env` in the working directory
3. Script queries OpenAI and Gemini concurrently (30-second timeout per API)
4. Returns JSON with each model's response
5. Claude synthesizes all three perspectives (ChatGPT + Gemini + its own) and presents a unified plan with inline attribution

## Setup

Copy `.env.template` to `.env` in the working directory:

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...

# Optional: override defaults
OPENAI_MODEL=gpt-5-nano
GEMINI_MODEL=gemini-3-flash-preview
```

**Never commit `.env` to git.** It is listed in `.gitignore`.

## Model Configuration

Default models are chosen for speed and cost-effectiveness. Override via `.env`.

| Provider | Default | Budget | High Quality | Premium |
|----------|---------|--------|--------------|---------|
| OpenAI | `gpt-5-nano` | `gpt-5-nano` | `gpt-5.2` | `gpt-5.2-pro` |
| Gemini | `gemini-3-flash-preview` | `gemini-2.5-flash` | `gemini-3-flash-preview` | `gemini-3-pro-preview` |

Each `/council` command makes exactly **2 API calls** (one per provider). Monitor usage in your OpenAI and Google AI Studio dashboards.

## SKILL.md — Authoring Conventions

`llm-council/SKILL.md` is the authoritative skill definition. It has YAML frontmatter:

```yaml
---
name: llm-council
description: Multi-LLM collaborative brainstorming and planning. Use when user
  explicitly requests consultation with multiple AI models (ChatGPT, Gemini, other
  LLMs) before presenting an implementation plan...
---
```

The `description` drives model invocation — it must be specific enough to distinguish this skill from a generic "brainstorm" request. Include the exact trigger phrases users type.

## query_llms.py Contract

- **Input:** positional string argument — the prompt text
- **Invocation:** `python3 llm-council/scripts/query_llms.py "<prompt>"`
- **Output:** JSON to stdout with per-model keys and response text
- **Error behavior:** if one API fails or times out, the script continues with the other and marks the failed model as unavailable in the JSON
- **Timeout:** 30 seconds per API call (hard-coded)
- **Dependencies:** reads from `.env` using `python-dotenv` (or equivalent); no other config files expected

## Error Handling Conventions

| Situation | Expected behavior |
|-----------|------------------|
| `.env` missing or keys absent | Inform user; provide setup instructions; do not attempt API calls |
| One API fails / times out | Note which model is unavailable; proceed with the other |
| Both APIs fail | Offer Claude-only analysis; explicitly state no external models were consulted |

## Skill Installation

Upload `llm-council.skill` (the ZIP) in Claude Code via the skills interface. Set up `.env` before first use.

## Development Workflow

When changing skill behavior:

1. Edit `llm-council/SKILL.md` — this is the single source of truth for skill instructions
2. Edit `llm-council/scripts/query_llms.py` if API interaction changes
3. Rebuild the installable bundle: `zip -r llm-council.skill llm-council/` (from repo root)
4. Update `README.md` for any user-facing behavior changes
5. Update `llm-council/references/setup.md` for any configuration changes
6. Commit and push

## Output Format

The skill instructs Claude to present the final plan naturally, weaving in model-attributed insights inline:

```
Based on consultation with ChatGPT and Gemini, here's the recommended approach:

[Plan body with inline references like "ChatGPT highlighted..." or "Gemini suggested..."]

Key contributions:
- ChatGPT: [brief summary]
- Gemini: [brief summary]
```

Do not fabricate model responses. If a model's API call failed, say so explicitly.
