# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**llm-council-skill** is a Claude Code skill that enables multi-model brainstorming: when invoked, Claude queries ChatGPT and Gemini for their perspectives, synthesizes insights from all three models, and presents a unified implementation plan with attribution.

## Repository Layout

```
llm-council-skill/
├── llm-council/
│   ├── SKILL.md                  # The skill instructions
│   ├── scripts/
│   │   └── query_llms.py         # Python script that calls OpenAI + Gemini APIs
│   └── references/
│       └── SETUP.md              # Detailed API setup instructions
├── llm-council.skill             # Alternative skill file (flat format)
├── README.md
└── LICENSE
```

## Setup

The skill requires API keys in a `.env` file in the working directory:

```bash
# Create from template
cp llm-council/.env.template .env

# Add your keys
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...

# Optional: override default models
OPENAI_MODEL=gpt-5-nano         # default
GEMINI_MODEL=gemini-3-flash-preview  # default
```

**Install Python dependency:**
```bash
pip install openai google-generativeai python-dotenv
```

## How the Skill Works

1. User invokes with phrases like "Consult the council:", "Ask ChatGPT and Gemini about...", "Get perspectives from other AIs on..."
2. Claude runs `scripts/query_llms.py` with the user's prompt
3. The script calls OpenAI and Gemini APIs in parallel (30s timeout each)
4. Claude analyzes both responses + its own analysis, synthesizes a plan
5. Output shows the synthesized plan + key contributions from each model

## query_llms.py

The script accepts a prompt as a command-line argument and prints structured output Claude can parse. If one API fails (timeout, key error), the script notes the failure and continues with the available response.

## Model Configuration

| Tier | OpenAI | Gemini |
|------|--------|--------|
| Budget | `gpt-5-nano` (default) | `gemini-2.5-flash` |
| Balanced | `gpt-5-mini` | `gemini-3-flash-preview` (default) |
| Premium | `gpt-5.2` | `gemini-3-pro-preview` |

Each `/council` invocation makes 2 API calls (one OpenAI, one Gemini). Monitor API usage dashboards to control costs.

## Key Conventions

- **The `.env` file must be in the working directory** where Claude Code is running — not the skill directory
- **API keys stay in `.env`** — never hardcode them in `query_llms.py` or `SKILL.md`
- **30-second timeout per API**: if a model is slow or down, the skill degrades gracefully
- **The skill is trigger-phrase activated**: it should only fire when users explicitly ask for multi-model consultation, not for every request
- **Attribution is mandatory**: always note which ideas came from which model in the synthesized output
- **No SDK lock-in**: `query_llms.py` uses the official OpenAI and Google SDKs — keep them updated

## Troubleshooting

- "API key not found": Confirm `.env` is in the current working directory (not home dir)
- API timeout: One model's response will be missing; Claude notes this and proceeds
- Both APIs fail: Claude falls back to its own analysis only
