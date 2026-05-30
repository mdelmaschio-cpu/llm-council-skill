# LLM Council Skill

Multi-LLM collaborative brainstorming skill for Claude. Queries ChatGPT (OpenAI) and Gemini before synthesizing a unified implementation plan — useful for architectural decisions, technical design, and any problem that benefits from diverse AI perspectives.

## Repository Structure

```
llm-council-skill/
├── llm-council.skill          # Packaged skill file (ZIP archive) — importable directly into Claude
├── README.md                  # Full setup, model options, and cost guidance
├── LICENSE                    # MIT
└── llm-council/
    ├── SKILL.md               # Core skill instructions (what Claude reads when invoked)
    ├── .env.template          # API key template — copy to .env in working directory
    ├── scripts/
    │   └── query_llms.py      # Queries OpenAI and Gemini APIs in sequence
    └── references/
        └── setup.md           # Detailed setup walkthrough
```

## How the Skill Works

**Trigger phrases:** "Consult the council", "Ask ChatGPT and Gemini", "Get perspectives from other AI models", "Consult with other LLMs"

When invoked:
1. Claude runs `llm-council/scripts/query_llms.py` against the working directory's `.env`
2. Each model's response is collected (30-second timeout per API; failures are non-fatal)
3. Claude synthesizes a unified plan with attribution to each model's contributions

## Required Setup

Create `.env` in the **working directory** (not the repo root):
```
OPENAI_API_KEY=your_openai_key
GEMINI_API_KEY=your_gemini_key
OPENAI_MODEL=gpt-5-nano              # optional, this is the default
GEMINI_MODEL=gemini-3-flash-preview  # optional, this is the default
```

Start from `llm-council/.env.template`. This file is git-ignored and must never be committed.

## Development Conventions

- **Skill behavior:** Edit `llm-council/SKILL.md`
- **API query logic:** Edit `llm-council/scripts/query_llms.py`
- **Rebuild distribution file:** `zip -r llm-council.skill llm-council/` (run from repo root)
- **No automated tests** — validate via live invocation
- **Python dependencies:** Keep minimal; use `requests` for HTTP, `python-dotenv` for `.env` loading

## SKILL.md Frontmatter

```yaml
---
name: llm-council
description: [trigger phrases — expand this to improve invocation coverage]
version: X.Y.Z
---
```

The `description` field drives skill selection. Add more trigger phrases here if the skill is not being invoked when expected.

## Model Tiers

Configure via `.env`:

| Tier | OpenAI | Gemini |
|------|--------|--------|
| Budget | `gpt-5-nano` (default) | `gemini-2.5-flash` |
| Balanced (default) | `gpt-5-nano` | `gemini-3-flash-preview` |
| High Quality | `gpt-5.2` | `gemini-3-flash-preview` |
| Premium | `gpt-5.2-pro` | `gemini-3-pro-preview` |

Each invocation makes 2 API calls (one OpenAI, one Gemini). Cost is the sum of both models.

## Distribution

The `llm-council.skill` file is a ZIP archive of the `llm-council/` directory. Rebuild it after any edits:

```bash
# From repo root
zip -r llm-council.skill llm-council/
```

Users install the skill by uploading `llm-council.skill` directly in Claude.

## AI Assistant Guidelines

- Do **not** edit `llm-council.skill` directly — it is a ZIP binary; always rebuild from source
- The `.env` file must **never be committed** — it is git-ignored by design
- When adding new model support, update both `query_llms.py` and `llm-council/references/setup.md`
- Prefer non-breaking changes to `.env.template` — existing users should not need to reconfigure
- If the skill is not being triggered, the `description` frontmatter needs more trigger phrases — do not add more body content to fix invocation issues
