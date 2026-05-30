# llm-council-skill

A Claude Code skill that enables collaborative brainstorming with multiple AI models (ChatGPT and Gemini) before presenting implementation plans. Queries external LLMs in parallel and synthesizes their perspectives.

## Repository Purpose

When Claude is about to make a significant architectural or implementation decision, this skill queries OpenAI and Google Gemini APIs concurrently, then synthesizes all perspectives before responding. Trigger phrases: "consult the council", "ask other models", "get perspectives from other AIs".

## Structure

```
llm-council-skill/
├── README.md
├── LICENSE
├── llm-council.skill          # Binary ZIP file (embedded skill definition)
└── llm-council/
    ├── SKILL.md               # Main skill definition and instructions
    ├── scripts/
    │   └── query_llms.py      # Multi-LLM orchestrator (213 lines)
    └── references/
        └── SETUP.md           # Detailed configuration guide
```

## How the Skill Works

1. Claude receives a user request matching the trigger phrases
2. The SKILL.md instructs Claude to run `query_llms.py` with the question
3. The script queries OpenAI Chat Completions and Google Gemini API in parallel (30s timeout each)
4. Results are returned as JSON with model attribution
5. Claude synthesizes all perspectives and attributes insights to each model

## query_llms.py Architecture

**CLI-first design**: The script prefers system-installed CLI tools over direct API calls:
1. Try `gemini` CLI → fall back to Gemini API (`GOOGLE_API_KEY`)
2. Try `codex` CLI → fall back to OpenAI API (`OPENAI_API_KEY`)

**Default models:**
- OpenAI: `gpt-4.1-nano` (cost-optimized)
- Google: `gemini-2.0-flash-preview` (cost-optimized)

Override via `.env` file:
```
OPENAI_MODEL=gpt-4o
GOOGLE_MODEL=gemini-2.0-pro
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=AIza...
```

**Output format:** JSON with `chatgpt` and `gemini` keys containing model responses.

## Setup

1. Copy `llm-council/` to `~/.claude/skills/llm-council/` (or let a plugin manager handle it)
2. Set API keys in `~/.claude/skills/llm-council/.env` (see `references/SETUP.md`)
3. Optionally install `codex-cli` and/or `gemini-cli` for CLI-first mode

## Technology Stack

- **Language:** Python 3, Markdown
- **HTTP:** `requests` library (standard pip install)
- **APIs:** OpenAI Chat Completions v1, Google Gemini API
- **No build step, no package.json, no virtualenv required**

## Development Workflow

There is no automated test suite. To test changes to `query_llms.py`:

```bash
# Run directly with a test prompt
python3 llm-council/scripts/query_llms.py "What are the tradeoffs of microservices vs monolith?"

# Check JSON output structure
python3 llm-council/scripts/query_llms.py "test" | python3 -m json.tool
```

Validate that:
- Both `chatgpt` and `gemini` keys appear in output
- Graceful degradation when one API is unavailable (other model's response still returned)
- Timeout (30s) is respected

## The .skill File

`llm-council.skill` is a ZIP archive containing the skill definition for distribution. To update it after editing `llm-council/`:

```bash
zip -r llm-council.skill llm-council/
```

Do not edit `llm-council.skill` directly — always edit files in `llm-council/` and repackage.

## Key Conventions

- Always attribute insights to the specific model that provided them in the synthesis
- If one API fails, proceed with the other rather than aborting
- The script must be runnable standalone (no Claude dependency) so it can be tested independently
- Keep default models at the cost-effective tier; document upgrade paths in SETUP.md
- Parallel queries are the whole point — never make them sequential
