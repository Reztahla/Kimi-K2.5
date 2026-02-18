# Kimi Coding K2.5 Skills (macOS Terminal)

Curate concise skills you want K2.5 to follow when assisting on macOS. Paste these into the system prompt or tools configuration for quick reuse.

- **Safe shell execution**: Prefer read-only commands first (`ls`, `cat`, `python -m compileall`). Avoid `sudo` unless explicitly allowed.
- **Python env hygiene**: Use `uv venv` + `source .venv/bin/activate`; install with `uv pip install ...`.
- **File ops**: Read before writing. When writing, back up files (e.g., `cp file file.bak`) and echo changes.
- **Diff-first fixes**: Request a patch/diff, then apply manually; keep changes minimal.
- **Network caution**: Only hit endpoints provided by the user; respect proxies like `HTTPS_PROXY`/`ALL_PROXY` if set.
- **Observability**: Ask to run targeted tests or linters, not whole suites, unless requested.
- **Output control**: Streamline to actionable code blocks; add rollback hints for any stateful change.
- **Tool choice**: Prefer built-ins (`python`, `grep`, `sed`, `awk`) over new installs. If new tools are needed, explain why.
- **Security**: Never handle secrets in plain text; prefer `.env` with `export $(cat .env | xargs)`.
