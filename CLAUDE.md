# CLAUDE.md — LLM Council Skill

This file documents the `llm-council-skill` repository for AI assistants. Read this before making any changes.

## Overview

`llm-council-skill` is a Claude Code skill that enables **multi-LLM collaborative brainstorming**. When invoked, it queries both OpenAI (ChatGPT) and Google Gemini for their perspectives on a user's question, then synthesizes a comprehensive implementation plan attributing insights from all three models (Claude + ChatGPT + Gemini).

The skill is packaged as a `.skill` file (zip archive) for upload into Claude, plus the source directory `llm-council/` that contains the actual skill instructions, Python script, and supporting files.

## Repository Structure

```
llm-council-skill/
├── LICENSE                        # Project license
├── README.md                      # User-facing documentation
├── llm-council.skill              # Installable skill archive (zip)
└── llm-council/                   # Skill source directory
    ├── SKILL.md                   # Core skill instructions (Claude reads this)
    ├── .env.template              # Template for required API key configuration
    ├── references/
    │   └── setup.md               # Detailed setup instructions
    └── scripts/
        └── query_llms.py          # Python script that calls OpenAI + Gemini APIs
```

## Key Files and Their Roles

### `llm-council/SKILL.md`
The primary instructions file that Claude reads when the skill is active. It defines:
- The skill's YAML frontmatter (`name: llm-council`, `description`, trigger phrases)
- The 4-step workflow: query external LLMs → analyze → synthesize → present
- Setup requirements and error handling behavior
- Output format conventions

### `llm-council/scripts/query_llms.py`
The Python script invoked by Claude during skill execution. It:
- Reads `OPENAI_API_KEY`, `GEMINI_API_KEY`, `OPENAI_MODEL`, and `GEMINI_MODEL` from a `.env` file in the working directory
- Makes parallel API calls to both OpenAI and Gemini
- Returns JSON-structured responses for Claude to parse
- Has a 30-second per-API timeout with graceful error handling

### `llm-council/.env.template`
Template the user copies to `.env` in their working directory. Required environment variables:
```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
OPENAI_MODEL=gpt-5-nano        # optional, this is the default
GEMINI_MODEL=gemini-3-flash-preview  # optional, this is the default
```

### `llm-council.skill`
A zip archive of the `llm-council/` directory. Users upload this file to Claude to install the skill. Do not edit this directly — it must be rebuilt from the source directory when changes are made.

## How to Use / Invoke the Skill

### Installation
1. Upload `llm-council.skill` to Claude (Claude Desktop or Claude Code)
2. Copy `llm-council/.env.template` to `.env` in the working directory where Claude runs
3. Add real API keys to `.env`

### Invocation Triggers
The skill activates on natural language phrases such as:
- `Consult the council: <question>`
- `Ask ChatGPT and Gemini what they think about <topic>`
- `Get perspectives from other AI models on <decision>`
- `Consult with other LLMs: <question>`

### Execution Flow
When triggered, Claude:
1. Runs `python3 scripts/query_llms.py "<user question>"`
2. Parses the JSON output containing ChatGPT and Gemini responses
3. Analyzes all three model perspectives (including its own)
4. Presents a synthesized plan with inline attributions

## Model Configuration

Models are configured via `.env`. Each call to the skill makes **2 API calls** (one to OpenAI, one to Gemini).

| Provider | Default Model | Alternatives (increasing capability/cost) |
|----------|--------------|-------------------------------------------|
| OpenAI | `gpt-5-nano` | `gpt-5-mini`, `gpt-5.2`, `gpt-5.2-pro` |
| Gemini | `gemini-3-flash-preview` | `gemini-2.5-flash-lite`, `gemini-2.5-flash`, `gemini-3-pro-preview` |

**Recommended configurations:**
- Budget: `gpt-5-nano` + `gemini-2.5-flash`
- Balanced (default): `gpt-5-nano` + `gemini-3-flash-preview`
- High quality: `gpt-5.2` + `gemini-3-flash-preview`
- Premium: `gpt-5.2-pro` + `gemini-3-pro-preview`

## Development Workflow

### Making Changes to Skill Logic
1. Edit `llm-council/SKILL.md` (skill instructions) or `llm-council/scripts/query_llms.py` (API logic)
2. Test manually by invoking the skill and verifying output
3. Rebuild the `.skill` archive: `zip -r llm-council.skill llm-council/` from the repo root
4. Verify the updated `.skill` file installs and works correctly in Claude

### Development Prerequisites
- Python 3.x available in the environment
- `openai` and `google-generativeai` Python packages (or equivalent installed in the environment)
- Valid `.env` file with both API keys present

### Error Scenarios

| Scenario | Behavior |
|----------|----------|
| `.env` missing or no API keys | Inform user, provide setup instructions |
| One API call fails | Note unavailable model, proceed with the other |
| Both API calls fail | Offer Claude-only analysis without external consultation |
| API timeout (>30s) | Fail that API, continue with available response |

## Conventions

- **`.env` location**: Must be in the **current working directory** where Claude is running, not the repo root
- **Script invocation**: Always use `python3 scripts/query_llms.py` (relative path from skill root)
- **Output format**: Synthesized plan with inline attributions (e.g., "ChatGPT highlighted...", "Gemini suggested...") followed by a brief per-model contribution summary
- **Skill archive naming**: `llm-council.skill` matches the `name` field in `SKILL.md` frontmatter
- **No hardcoded keys**: API keys must always come from `.env`, never hardcoded

## Testing and Validation

There is no automated test suite. Manual validation steps:

1. **Env check**: Confirm `.env` exists with valid keys before testing
2. **Script smoke test**: Run `python3 llm-council/scripts/query_llms.py "test question"` directly and verify JSON output
3. **Skill invocation test**: Install the `.skill` file in Claude and trigger with a real prompt
4. **Error path test**: Test with a missing or invalid API key to verify graceful error messages
5. **Model switch test**: Change `OPENAI_MODEL` / `GEMINI_MODEL` in `.env` and confirm the new model is used

## Related Resources

- OpenAI API keys: https://platform.openai.com/api-keys
- Gemini API keys: https://aistudio.google.com/app/apikey
- Claude Code docs: https://docs.anthropic.com/en/docs/claude-code
