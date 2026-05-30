# CLAUDE.md — llm-council-skill

This file provides guidance to Claude Code when working in this repository.

## Project Purpose

**LLM Council** is a single Claude Code skill that enables multi-model collaborative brainstorming. When a user explicitly asks Claude to consult with other AI models, this skill queries both the OpenAI API (ChatGPT) and the Google Gemini API in parallel, analyzes their responses, and synthesizes a comprehensive implementation plan attributing insights from all three models (Claude + ChatGPT + Gemini).

This is a focused, single-skill repository. The core artifact is the `llm-council.skill` zip file (for direct installation) and the `llm-council/` directory containing the skill content.

## Repository Structure

```
llm-council-skill/
├── README.md                   # Installation, model options, API costs, troubleshooting
├── LICENSE                     # MIT
├── llm-council.skill           # Packaged skill zip (for direct install)
│
└── llm-council/                # Skill content directory
    ├── SKILL.md                # Main skill instructions and workflow
    ├── .env.template           # Template for API key configuration
    ├── scripts/
    │   └── query_llms.py       # Python script — queries OpenAI and Gemini APIs in parallel
    └── references/
        └── setup.md            # Detailed setup instructions
```

## Skill Activation

The skill activates when the user uses any of these phrases:
- "Consult the council: ..."
- "Ask ChatGPT and Gemini about ..."
- "Get perspectives from other AI models on ..."
- "Consult with other LLMs: ..."

The `description` field in `llm-council/SKILL.md` drives model skill-selection. It must remain trigger-phrase-rich — do not replace it with a generic description.

## Workflow

When invoked, the skill follows this process:

1. **Query external LLMs** — Execute `python3 scripts/query_llms.py "<user_prompt>"` to get JSON responses from both ChatGPT and Gemini
2. **Parse responses** — Extract each model's perspective from the JSON output
3. **Synthesize plan** — Create an implementation plan incorporating the best ideas from all three models (Claude's own analysis + ChatGPT contributions + Gemini contributions)
4. **Present with attribution** — Show the final plan with inline references to which model contributed which insight

## Configuration

API keys and model preferences are stored in a `.env` file in the working directory (not committed to git):

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...

# Optional — defaults shown
OPENAI_MODEL=gpt-5-nano
GEMINI_MODEL=gemini-3-flash-preview
```

Copy `.env.template` to `.env` and fill in the keys. The `.env` file must be in the **current working directory** when the skill runs, not in the skill directory itself.

## Supported Models

### OpenAI (ordered by capability)
| Model | Cost Tier | Notes |
|-------|-----------|-------|
| `gpt-5-nano` | Very low | Default — fastest, most cost-efficient |
| `gpt-5-mini` | Low-moderate | Balanced cost and quality |
| `gpt-5.2` | Moderate-high | Best for coding and agentic tasks |
| `gpt-5.2-pro` | High | Professional knowledge work |

### Gemini (ordered by capability)
| Model | Cost Tier | Notes |
|-------|-----------|-------|
| `gemini-2.5-flash-lite` | Very low | Ultra-fast, high throughput |
| `gemini-2.5-flash` | Low | Best price-performance |
| `gemini-3-flash-preview` | Moderate | Default — balanced |
| `gemini-3-pro-preview` | High | Best reasoning, complex tasks |

**Recommended configurations:**
- **Budget**: `gpt-5-nano` + `gemini-2.5-flash`
- **Balanced** (default): `gpt-5-nano` + `gemini-3-flash-preview`
- **High quality**: `gpt-5.2` + `gemini-3-flash-preview`
- **Premium**: `gpt-5.2-pro` + `gemini-3-pro-preview`

Each invocation makes exactly 2 external API calls — one to OpenAI, one to Gemini.

## The query_llms.py Script

`llm-council/scripts/query_llms.py` is the only executable component. It:
- Accepts the user's prompt as a command-line argument
- Queries both APIs concurrently (30-second timeout per API)
- Outputs JSON with each model's response and metadata
- Handles partial failures gracefully — if one API fails, returns the other's response plus an error note

**Error behavior:**
- Missing API key → informs user and provides setup instructions, does not crash
- API timeout → notes which model timed out, continues with available responses
- Both APIs fail → informs user and offers to provide Claude's own analysis without consultation

## SKILL.md Format

The `llm-council/SKILL.md` follows the standard Claude Code skill format:

```yaml
---
name: llm-council
description: Multi-LLM collaborative brainstorming... [trigger-phrase-rich]
---

# Skill Title

[Workflow, setup requirements, usage examples, output format, error handling]
```

The frontmatter allows only `name` and `description` as fields. Do not add `version`, `author`, `license`, or other metadata.

## Output Format

The skill presents output naturally in prose, not as a rigid template. Key conventions:

- Mention key insights from ChatGPT and Gemini **inline** within the plan, not just in a trailing summary
- End with a "Key contributions" section listing each model's main contribution in 1–2 sentences
- If one model's perspective was unavailable, note this clearly and explain why
- Never fabricate responses from a model that failed or was not queried

Example structure:
```
Based on consultation with ChatGPT and Gemini, here's the recommended approach:

[Implementation plan with inline references like "ChatGPT highlighted..." or "Gemini suggested..."]

Key contributions:
- ChatGPT: [brief summary of their unique input]
- Gemini: [brief summary of their unique input]
```

## Development Workflow

### Editing the Skill

- Core workflow is in `llm-council/SKILL.md` — edit this to change how Claude orchestrates the consultation
- `llm-council/scripts/query_llms.py` handles external API calls — edit this for API changes, new model support, or output format changes
- The `llm-council.skill` zip should be regenerated after changes to the `llm-council/` directory

### Regenerating the .skill Package

```bash
cd llm-council-skill/
zip -r llm-council.skill llm-council/
```

### Testing Locally

1. Copy `.env.template` to `.env` in your working directory and add API keys
2. Run the script directly to verify connectivity:
   ```bash
   python3 llm-council/scripts/query_llms.py "What are the best approaches for microservices architecture?"
   ```
3. Verify JSON output contains responses from both models

### Adding a New Model Provider

To add support for a third LLM (e.g., Mistral):
1. Add the API key variable to `.env.template`
2. Add query logic to `query_llms.py` (parallel with existing calls)
3. Update `SKILL.md` setup requirements and model options tables
4. Update README.md model options section
5. Update the output format to attribute the new model

## Important Files

| File | Role |
|------|------|
| `llm-council/SKILL.md` | Main skill — orchestration workflow, setup requirements, output format |
| `llm-council/scripts/query_llms.py` | External API caller — queries OpenAI and Gemini in parallel |
| `llm-council/.env.template` | Configuration template — copy to `.env` and add keys |
| `llm-council/references/setup.md` | Detailed setup instructions for first-time users |
| `llm-council.skill` | Packaged zip for direct installation |

## How AI Assistants Should Behave Here

- **Never fabricate model responses** — only report what the APIs actually returned
- **Preserve the trigger phrases** in `SKILL.md`'s `description` field — they drive skill invocation
- **Handle API failures gracefully** — always proceed with available responses rather than failing completely
- **Keep the skill single-purpose** — this skill does one thing (multi-model consultation) and should not expand to do unrelated tasks
- **Document model costs accurately** — when updating model options, include accurate cost information so users can make informed choices
- **Test with real API keys** before packaging a new `.skill` file
- **Regenerate `llm-council.skill`** after any changes to the `llm-council/` directory
