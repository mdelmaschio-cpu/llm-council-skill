# llm-council-skill

A Claude Code skill that consults external LLMs (ChatGPT + Gemini) to get independent perspectives on a problem, then synthesizes the results.

## Repository Purpose

Provides a `/llm-council` skill for Claude Code that queries OpenAI and Google Gemini APIs in parallel, returns both responses as structured JSON, and lets Claude synthesize a consensus or highlight disagreements. Useful for triangulating architectural decisions, getting second opinions, and de-biasing Claude's own recommendations.

## Structure

```
llm-council-skill/
├── README.md                    # User-facing docs with install instructions
├── LICENSE                      # MIT, Copyright 2026 Gustavo Publio
├── llm-council.skill            # Packaged zip for distribution (DO NOT manually edit)
└── llm-council/                 # Source directory
    ├── SKILL.md                 # Skill definition (YAML frontmatter + instructions)
    ├── .env.template            # Environment variable template
    ├── scripts/
    │   └── query_llms.py        # Core Python execution script (212 lines)
    └── references/
        └── SETUP.md             # Detailed setup and configuration guide
```

## How the Skill Works

1. Claude receives a user request prefixed with a trigger phrase (e.g., "Consult the council:", "Ask ChatGPT and Gemini").
2. Claude invokes the skill, which runs `python3 scripts/query_llms.py "<prompt>"`.
3. The script queries OpenAI and Gemini in sequence, outputting JSON with both responses.
4. Claude reads the JSON and synthesizes an implementation plan, noting agreements and conflicts.

**Output JSON structure:**
```json
{
  "prompt": "user prompt",
  "chatgpt": { "model": "gpt-5-nano", "source": "api", "response": "..." },
  "gemini":  { "model": "gemini-3-flash-preview", "source": "api", "response": "..." }
}
```

## Environment Setup

Copy `.env.template` to `.env` in the project root and populate:

```
OPENAI_API_KEY=sk-...          # Required - platform.openai.com/api-keys
GEMINI_API_KEY=AIza...         # Required - aistudio.google.com/app/apikey
OPENAI_MODEL=gpt-5-nano        # Optional - defaults to gpt-5-nano
GEMINI_MODEL=gemini-3-flash-preview  # Optional
```

The script loads `.env` from the current working directory, falling back to OS environment variables.

## Model Configuration

| Tier | OpenAI | Gemini | Use case |
|------|--------|--------|----------|
| Cost-efficient (default) | gpt-5-nano | gemini-3-flash-preview | Daily use |
| Balanced | gpt-5-mini | gemini-2.5-flash | Better quality |
| High quality | gpt-5.2 | gemini-3-pro-preview | Critical decisions |
| Maximum | gpt-5.2-pro | gemini-3-pro-preview | Highest stakes |

## Script Architecture (`scripts/query_llms.py`)

**Execution priority:**
1. CLI tools (`gemini` or `codex` in PATH) — preferred, more token-efficient
2. REST API calls — fallback when CLIs unavailable
3. Error message — if neither available

**Key functions:**
- `load_env_file()` — parses `.env` key=value pairs from CWD
- `is_cli_available(name)` — checks PATH for CLI tool
- `query_openai(prompt, api_key, model)` — POST to OpenAI chat completions, 30s timeout, max 2000 tokens
- `query_gemini(prompt, api_key, model)` — POST to Google GenerativeLanguage API, 30s timeout, max 2000 tokens
- `query_gemini_cli(prompt)` / `query_codex_cli(prompt)` — subprocess with 60s timeout

**Error handling:** If one API fails, the script continues and reports an error in that provider's response field. Claude handles graceful degradation.

## Testing the Script

```bash
cd llm-council
python3 scripts/query_llms.py "What is the best way to handle errors in Python?"
```

Expects JSON output. Check `source` field: `"api"` or `"cli"`.

## Packaging

The `llm-council.skill` file is a zip archive of the `llm-council/` directory. Rebuild it after source changes:

```bash
cd llm-council-skill
zip -r llm-council.skill llm-council/
```

Users install by uploading `llm-council.skill` to Claude Code.

## Trigger Phrases

The skill is invoked by any of:
- "Consult the council: [question]"
- "Ask ChatGPT and Gemini: [question]"
- "Get a second opinion on: [question]"
- "What do other AIs think about: [question]"

## Development Conventions

- **No package.json / no npm** — Python stdlib only (`os`, `sys`, `json`, `shutil`, `subprocess`, `urllib.request`)
- **Standard library HTTP** — uses `urllib.request`, not `requests`, to avoid dependencies
- **Environment variables**: UPPERCASE_WITH_UNDERSCORES
- **Python files**: lowercase_with_underscores
- **No test suite** — manual testing via CLI invocation
- **No CI pipeline** — pure documentation and script repository
