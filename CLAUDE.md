# CLAUDE.md — llm-council-skill

This file provides guidance to AI assistants working in this repository.

## Repository Purpose

A single Claude skill that enables collaborative AI brainstorming by querying both ChatGPT (OpenAI) and Gemini (Google) before synthesizing a unified implementation plan. Designed for decisions where multiple AI perspectives add value: architecture choices, technical trade-offs, complex algorithms.

## Repository Structure

```
llm-council-skill/
├── README.md                   # Installation, model options, cost guide
├── LICENSE                     # MIT
└── llm-council/                # The skill package (installable directory)
    ├── SKILL.md                # Skill instructions loaded by Claude
    ├── .env.template           # API key configuration template
    ├── references/
    │   └── setup.md            # Detailed setup walkthrough
    └── scripts/
        └── query_llms.py       # Python script: calls OpenAI + Gemini APIs in parallel
```

## How the Skill Works

1. User triggers with phrases like "Consult the council:" or "Ask ChatGPT and Gemini..."
2. Claude runs `llm-council/scripts/query_llms.py` which calls both APIs
3. Claude analyzes each model's response and identifies unique insights
4. Claude synthesizes a final plan, attributing each model's contribution

## Installation

```bash
# Claude Code (global)
cp -r llm-council ~/.claude/skills/llm-council

# Configure API keys
cp llm-council/.env.template .env
# Then edit .env with your keys
```

## Configuration (`.env` file)

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=AIza...
OPENAI_MODEL=gpt-5-nano          # default: cost-effective
GEMINI_MODEL=gemini-3-flash-preview  # default: balanced
```

Model tiers:
- **Budget:** `gpt-5-nano` + `gemini-2.5-flash` — routine brainstorming
- **Balanced (default):** `gpt-5-nano` + `gemini-3-flash-preview`
- **Premium:** `gpt-5.2` + `gemini-3-pro-preview` — critical decisions

## Invocation Phrases

- `"Consult the council: How should I..."`
- `"Ask ChatGPT and Gemini what they think about..."`
- `"Get perspectives from other AI models on..."`
- `"Consult with other LLMs: What's the best approach for..."`

## Development Conventions

- The skill folder name (`llm-council`) must match the `name` field in `SKILL.md`
- `query_llms.py` has a 30-second timeout per API call — do not increase without justification
- If one API fails, the skill proceeds with available responses and notes the failure
- The `.env` file is gitignored — never commit API credentials
- No additional Python dependencies beyond `openai` and `google-generativeai` packages

## Testing

```
# In a Claude Code session with the skill installed:
Consult the council: How should I structure a REST API for a multi-tenant SaaS app?
```

Expected behavior: Claude queries both APIs, shows each model's response, synthesizes a final plan with attribution.

## Important Notes for AI Assistants

- Do NOT hardcode API keys anywhere in the codebase — always use `.env`
- The 30-second timeout is intentional; if an API is slow, the skill logs it and continues
- Model names in README may become outdated as providers release new versions — check the README before suggesting specific model names
- Each invocation makes exactly 2 API calls (one OpenAI, one Gemini) — factor cost accordingly
- The `.env.template` file shows the structure but contains no real keys — safe to commit
