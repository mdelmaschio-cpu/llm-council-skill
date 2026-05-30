# CLAUDE.md — llm-council-skill

A Claude Code skill that queries ChatGPT (OpenAI) and Gemini (Google) before synthesizing an implementation plan. Gives Claude access to perspectives from multiple AI models in a single conversation.

## What This Repo Is

When the user says "consult the council" or "ask other AI models", Claude runs `llm-council/scripts/query_llms.py`, receives JSON responses from ChatGPT and Gemini, synthesizes the insights, and presents a combined implementation plan with attribution.

## Repository Structure

```
llm-council-skill/
├── README.md
├── LICENSE
├── llm-council.skill          # Installable skill file (upload this to Claude Code)
└── llm-council/               # Skill contents
    ├── SKILL.md               # Main instructions for Claude — triggers, workflow, output format
    ├── .env.template          # Copy to .env and fill in API keys
    ├── scripts/
    │   └── query_llms.py      # Python script that queries both LLM APIs
    └── references/
        └── SETUP.md           # Detailed API key setup guide
```

### Key Files

**`llm-council.skill`** — The file users upload to Claude Code to install the skill. Contains the packaged skill definition.

**`llm-council/SKILL.md`** — Claude's instructions. Defines the trigger phrases, 4-step workflow, setup requirements, output format, and error handling. This is what Claude reads when the skill is invoked.

**`llm-council/scripts/query_llms.py`** — Python script that queries both APIs and outputs JSON. Claude calls this with `python3 scripts/query_llms.py "<prompt>"`.

**`llm-council/.env.template`** — Template for the required `.env` configuration file (never committed; users create their own).

## Skill Invocation

Trigger phrases (any of these activate the skill):
- "Consult the council: ..."
- "Ask ChatGPT and Gemini what they think about ..."
- "Get perspectives from other AI models on ..."
- "Consult with other LLMs: ..."

**Claude's 4-step process:**
1. Run `python3 scripts/query_llms.py "<user prompt>"` — gets perspectives from ChatGPT and Gemini
2. Parse JSON response — extract each model's suggestions
3. Synthesize — combine the best ideas from all three models (ChatGPT + Gemini + Claude's own analysis)
4. Present — show the final plan with inline attribution to each model's contributions

## `query_llms.py` — How It Works

The script uses a **CLI-first, API-fallback** strategy:

| Model | First attempt | Fallback |
|---|---|---|
| ChatGPT | `codex -p "<prompt>"` (OpenAI Codex CLI) | OpenAI REST API with `OPENAI_API_KEY` |
| Gemini | `gemini -p "<prompt>"` (Gemini CLI) | Google Generative Language API with `GEMINI_API_KEY` |

**Output format** (always JSON on stdout):

```json
{
  "prompt": "...",
  "chatgpt": {
    "model": "gpt-5-nano",
    "source": "api (gpt-5-nano)",
    "response": "..."
  },
  "gemini": {
    "model": "gemini-3-flash-preview",
    "source": "api (gemini-3-flash-preview)",
    "response": "..."
  }
}
```

**Timeouts:** 60 seconds per CLI call, 30 seconds per API call. If one model fails, the script continues and returns an error string in that model's `response` field.

## Configuration

Users must create a `.env` file in their working directory. Template:

```
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...

# Optional — defaults shown
OPENAI_MODEL=gpt-5-nano
GEMINI_MODEL=gemini-3-flash-preview
```

The script loads `.env` from the current working directory, then falls back to environment variables.

### Model Options

**OpenAI** (ascending capability/cost):
- `gpt-5-nano` — default, fastest, cheapest
- `gpt-5-mini` — balanced
- `gpt-5.2` — best for coding and complex reasoning
- `gpt-5.2-pro` — highest capability, highest cost

**Gemini** (ascending capability/cost):
- `gemini-2.5-flash-lite` — ultra-fast, throughput-optimized
- `gemini-2.5-flash` — best price-performance
- `gemini-3-flash-preview` — default, balanced
- `gemini-3-pro-preview` — most intelligent, best reasoning

## Installation

1. Upload `llm-council.skill` to Claude Code
2. Copy `.env.template` to `.env` in the working directory
3. Fill in `OPENAI_API_KEY` and `GEMINI_API_KEY`
4. Optionally set `OPENAI_MODEL` and `GEMINI_MODEL`

To verify: `python3 llm-council/scripts/query_llms.py "Test prompt"` — should output valid JSON.

## Development Workflow

### Git Branches

- Feature development branch: `claude/claude-md-docs-nI6a7`
- Push to that branch, open PR to `main`

### Making Changes

**`SKILL.md` changes** — affect Claude's behavior. Test by:
1. Installing the updated skill
2. Issuing a "consult the council" prompt
3. Verifying Claude follows the new instructions

**`query_llms.py` changes** — test directly:
```bash
python3 llm-council/scripts/query_llms.py "your test prompt"
```
Verify the JSON output structure matches what `SKILL.md` documents Claude should parse.

**Model default changes** — update both `SKILL.md` (Setup Requirements section), `.env.template`, and `README.md` to stay consistent.

### Dependencies

The script requires only the Python standard library plus `requests`. No `requirements.txt` exists; ensure `requests` is available:

```bash
pip install requests
```

CLI tools (`gemini`, `codex`) are optional but preferred — they avoid needing API keys.

## Error Handling

- **Missing `.env`** — script returns error string in the response field; Claude should inform the user and provide setup instructions from `references/SETUP.md`
- **API call failure** — script returns error string; Claude proceeds with available responses and notes which model was unavailable
- **Both APIs fail** — Claude offers its own analysis without external consultation
- **CLI unavailable** — automatic fallback to API; transparent via `source` field in JSON output

## Key Conventions for AI Assistants

1. **Always run `query_llms.py` before presenting the plan** — do not skip the external consultation step.
2. **Parse the full JSON response** — check both `chatgpt.response` and `gemini.response` for error strings before synthesizing.
3. **Attribution is mandatory** — the output format requires inline references and a "Key contributions" summary.
4. **Never commit `.env`** — it contains real API keys. The `.gitignore` should cover this, but double-check.
5. **Model defaults in three places** — `SKILL.md`, `.env.template`, and `README.md` must stay in sync when defaults change.
