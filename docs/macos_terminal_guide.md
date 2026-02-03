# Kimi Coding K2.5 – macOS Terminal Implementation Guide

This guide walks through using the Kimi Coding K2.5 model from a macOS terminal. It assumes you will **consume** a deployment (e.g., vLLM or SGLang) that already serves the model with `--tool-call-parser kimi_k2` and `--reasoning-parser kimi_k2` enabled.

## Prerequisites
- macOS 13+ (Apple Silicon recommended)
- Homebrew installed
- Python 3.10+ available as `python3`
- A reachable Kimi Coding K2.5 inference endpoint (OpenAI-compatible HTTP API) and API key

## Quick setup (one-time)
```bash
# Install uv (fast Python package manager)
brew install uv

# Create a project folder
mkdir -p ~/kimi-k25 && cd ~/kimi-k25

# Create a virtual environment and install client deps
uv venv
source .venv/bin/activate
uv pip install "openai>=1.60.0" httpx python-dotenv
```

Create a `.env` file so your keys are not echoed in shell history:
```bash
cat <<'EOF' > .env
KIMI_BASE_URL=https://your.kimi.endpoint/v1
KIMI_API_KEY=sk-xxxxx
KIMI_MODEL=kimi-coding-k2.5
EOF
```

Load the environment when working:
```bash
source .venv/bin/activate
export $(cat .env | xargs)
```

## Call the model from macOS terminal
### Minimal chat request (Python)
```bash
python - <<'PY'
import os
from openai import OpenAI

client = OpenAI(base_url=os.environ["KIMI_BASE_URL"], api_key=os.environ["KIMI_API_KEY"])

resp = client.chat.completions.create(
    model=os.environ["KIMI_MODEL"],
    messages=[
        {"role": "system", "content": "You are Kimi Coding K2.5, a macOS terminal pair programmer."},
        {"role": "user", "content": "Create a one-file Python HTTP server that returns hello world."},
    ],
    temperature=0.2,
)

print(resp.choices[0].message.content)
PY
```

### With tool use (macOS-safe)
Provide tools you allow K2.5 to call; run them locally only after inspecting the JSON tool calls.
```bash
python - <<'PY'
import json, os
from openai import OpenAI

client = OpenAI(base_url=os.environ["KIMI_BASE_URL"], api_key=os.environ["KIMI_API_KEY"])

tools = [
    {
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read a UTF-8 text file from the working directory",
            "parameters": {
                "type": "object",
                "properties": {"path": {"type": "string"}},
                "required": ["path"],
            },
        },
    },
]

resp = client.chat.completions.create(
    model=os.environ["KIMI_MODEL"],
    messages=[
        {"role": "system", "content": "You are Kimi Coding K2.5. Use tools when you need filesystem context."},
        {"role": "user", "content": "Inspect main.py and suggest a fix if imports fail."},
    ],
    tools=tools,
    tool_choice="auto",
)

print(json.dumps(resp.choices[0].message.model_dump(), indent=2))
PY
```

### cURL sanity check
```bash
curl -X POST "$KIMI_BASE_URL/chat/completions" \
  -H "Authorization: Bearer $KIMI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$KIMI_MODEL"'",
    "messages": [
      {"role":"system","content":"You are Kimi Coding K2.5 on macOS terminal."},
      {"role":"user","content":"Give me a single bash command to print python version."}
    ],
    "temperature": 0.1
  }'
```

## macOS-friendly defaults
- Keep shell commands **idempotent**: prefer `ls`, `cat`, `python -m pip install --dry-run` when exploring.
- Avoid sudo unless required; prefer Homebrew user installs.
- When requesting code execution, ask K2.5 for a **command block plus a brief rollback note** (e.g., how to delete created files).
- For large outputs, ask for `--output-file` suggestions or `tee` usage.

## Troubleshooting
- TLS or certificate errors: update Homebrew OpenSSL (`brew install openssl@3`) and set `SSL_CERT_FILE` if needed.
- venv not activating: ensure `source .venv/bin/activate` and that the shell is `bash`/`zsh`.
- Tool calls look unsafe: do not execute blindly; run in a temp directory and add guardrails to the tool list.
