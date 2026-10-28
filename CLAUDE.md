# {PROJECT_NAME} — Instructions for AI Coding Assistants

Read [.github/copilot-instructions.md](.github/copilot-instructions.md) (playbook) fully before non-trivial changes. Known mistakes: [docs/COPILOT_PITFALLS_COOKBOOK.md](docs/COPILOT_PITFALLS_COOKBOOK.md).

## Memory: the repo is the only memory

- Do NOT use private (tool-side) memory. Write every rule, decision, preference and correction — **with its WHY** — into the repo in the same turn: cross-cutting → playbook; detail/history → `docs/*.md`; mistakes → cookbook; open work → `plans/`.
- Exception, only if the info is unrelated to the project or cannot be committed (secret, personal data) — and the repo doesn't cover it. Doubt → repo. Never a valid reason: "session-specific", "just rationale", "small preference".
- If you used private memory, tell the user that turn (file + reason). At session start, move any project knowledge found there into the repo and delete the copy.
- Rule text lives in the playbook/docs, not here.
