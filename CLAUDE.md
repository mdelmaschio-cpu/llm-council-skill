# CLAUDE.md — LLM Council Skill

## Repository Overview

This repository contains a **Claude Code skill** called `llm-council` that enables Claude to collaborate with other AI models (ChatGPT and Gemini) before presenting implementation plans. When a user asks Claude to "consult the council", Claude runs a Python script that queries both external LLM APIs, then synthesizes a unified response incorporating all three perspectives.

The primary deliverable is `llm-council.skill` — a ZIP archive that can be uploaded directly to Claude Code as an installable skill.

---

## Directory Structure

```
llm-council-skill/
├── llm-council.skill          # Distributable skill archive (ZIP format)
├── llm-council/               # Source files included in the skill archive
│   ├── SKILL.md               # Skill instructions Claude reads at runtime
│   ├── .env.template          # Template for required API keys
│   ├── scripts/
│   │   └── query_llms.py      # Python script that queries OpenAI and Gemini APIs
│   └── references/
│       └── SETUP.md           # End-user setup guide for API keys and configuration
├── README.md                  # Public-facing documentation with usage and install info
└── LICENSE                    # License file
```

### Two parallel representations of the same content

- `llm-council/` — The source directory containing all skill files in plain text (editable)
- `llm-council.skill` — A ZIP archive packaging those same files for distribution; this is what users actually install

When modifying the skill, edit the files inside `llm-council/`, then re-package them into `llm-council.skill`. The two must stay in sync.

---

## Key Files and Their Roles

### `llm-council/SKILL.md`
The core skill definition. Claude reads this file at runtime when the skill is active. It contains:
- A YAML front matter block (`name`, `description`) that tells Claude Code when to activate the skill
- Step-by-step workflow instructions Claude follows when invoked
- The exact `python3 scripts/query_llms.py "<prompt>"` command Claude should run
- Output format guidance for presenting the synthesized response
- Error handling instructions for missing keys or failed API calls

### `llm-council/scripts/query_llms.py`
Standalone Python 3 script that queries both external LLMs and prints JSON to stdout.

**Key behavior:**
- Accepts the user prompt as CLI arguments: `python3 query_llms.py "your question here"`
- Priority: tries CLI tools (`gemini` binary, `codex` binary) first; falls back to direct REST API calls
- Loads `.env` from the current working directory (where Claude is running), with `os.environ` as fallback
- Outputs structured JSON: `{ "prompt": ..., "chatgpt": { "model", "source", "response" }, "gemini": { ... } }`
- Per-API timeout: 30 seconds for API calls, 60 seconds for CLI calls
- Only dependency beyond stdlib: `requests`

### `llm-council/.env.template`
Documents the four environment variables the script reads:
- `OPENAI_API_KEY` — required unless `codex` CLI is installed
- `GEMINI_API_KEY` — required unless `gemini` CLI is installed
- `OPENAI_MODEL` — optional, defaults to `gpt-5-nano`
- `GEMINI_MODEL` — optional, defaults to `gemini-3-flash-preview`

### `llm-council/references/SETUP.md`
Step-by-step guide for end users covering API key acquisition, `.env` file creation, model selection, and testing. Included in the skill archive so Claude can reference it when helping users who encounter setup issues.

### `llm-council.skill`
ZIP archive containing the same files as `llm-council/`. The file paths inside the archive mirror the directory structure. Inspect contents with `unzip -l llm-council.skill`.

---

## Development Setup

### Prerequisites
- Python 3.x
- `requests` library: `pip install requests`

### Testing the Script Directly
```bash
# From the llm-council/ directory (so .env is found)
cd llm-council
cp .env.template .env
# Edit .env with real API keys
python3 scripts/query_llms.py "How should I structure a microservices architecture?"
```

Expected output is a JSON object with `chatgpt` and `gemini` keys, each containing `model`, `source`, and `response` fields.

### Repackaging the Skill Archive
After editing source files in `llm-council/`, rebuild the `.skill` archive:
```bash
cd llm-council
zip -r ../llm-council.skill SKILL.md .env.template scripts/ references/
```

The archive must be built from inside the `llm-council/` directory so paths inside the ZIP don't include the `llm-council/` prefix.

### Installing the Skill in Claude Code
1. In Claude Code settings, navigate to Skills
2. Upload `llm-council.skill`
3. Create a `.env` file in your working directory with your API keys

---

## Skill Conventions and Patterns

### Skill Activation (SKILL.md front matter)
```yaml
---
name: llm-council
description: Multi-LLM collaborative brainstorming and planning. Use when user explicitly
  requests consultation with multiple AI models...
---
```
The `description` field is what Claude Code uses to decide when to invoke this skill. Keep it specific to avoid false positives — this skill should only activate on explicit requests like "consult the council" or "ask ChatGPT and Gemini".

### Trigger Phrases
The skill is designed to activate on:
- "Consult the council: ..."
- "Ask ChatGPT and Gemini ..."
- "Get perspectives from other AI models ..."
- "Consult with other LLMs: ..."

### Script Output Contract
`query_llms.py` always prints valid JSON to stdout (never crashes silently). Claude parses this JSON, extracts responses, and synthesizes them. The `source` field in each response indicates whether the data came from a CLI tool (`codex-cli`, `gemini-cli`) or the REST API (e.g., `api (gpt-5-nano)`), which is useful for debugging.

### CLI-First Design
The script prefers CLI tools (`codex`, `gemini`) over direct API calls when they are available in `PATH`. This is intentional — CLI tools can be more token-efficient and may use pre-configured auth. API keys in `.env` serve as fallbacks.

### Model Configuration
Both models are configurable via `.env` without code changes. The defaults (`gpt-5-nano` + `gemini-3-flash-preview`) favor speed and cost. Users doing serious architectural work should upgrade to `gpt-5.2` + `gemini-3-pro-preview`.

---

## Notes for AI Assistants

### What this skill does at runtime
When invoked, Claude (with this skill active) will:
1. Call `python3 scripts/query_llms.py "<user prompt>"` as a bash command
2. Parse the returned JSON
3. Read both `chatgpt.response` and `gemini.response`
4. Synthesize a plan that credits insights from each model
5. Present inline attributions (e.g., "ChatGPT highlighted..." / "Gemini suggested...")

### Modifying the skill workflow
Edit `llm-council/SKILL.md` — this is the only file Claude reads for runtime instructions. Changes to `query_llms.py` affect what data Claude receives; changes to `SKILL.md` affect how Claude uses that data.

### The .env location matters
The script calls `load_env_file(".env")` — a relative path. Claude must run the script from the directory where the user's `.env` lives (typically the project root / Claude's working directory). If the script can't find `.env`, it falls back to environment variables, then emits an error string in the response field (not a raised exception).

### Error strings, not exceptions
When an API call fails or a key is missing, `query_llms.py` puts a human-readable error string in the `response` field of the JSON output rather than raising an exception or exiting non-zero. Claude's error handling in `SKILL.md` is written accordingly: check if the response looks like an error and inform the user gracefully.

### Keeping source and archive in sync
A common mistake is editing `llm-council/SKILL.md` (the source) but forgetting to rebuild `llm-council.skill` (the archive). The installed skill reads only from the archive. After any source edit, the archive must be rebuilt and reinstalled.

### No test suite
There are no automated tests. Manual testing via `python3 scripts/query_llms.py "test prompt"` is the only verification path.

### Python dependencies
The only non-stdlib dependency is `requests`. No `requirements.txt` exists; install it manually with `pip install requests` if missing.
