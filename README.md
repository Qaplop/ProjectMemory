# {PROJECT_NAME}

> **About this repo** (delete this block in your project)
> Technology-neutral boilerplate of rules, workflows and pitfalls for AI-assisted coding projects: a playbook, a pitfalls cookbook and a README template.
> **New project:** copy `CLAUDE.md`, `README.md`, `.github/` and `docs/` into it → replace every `{PLACEHOLDER}` → delete rules that don't apply → add project rules in the same DO / DON'T / WHY shape.

{One paragraph: what it does, for whom, main value.}

## Requirements

- Toolchain: {version}
- Dependencies: {name — version}
- Platforms/targets: {list}

## Features

- **{Area}**: {capability} — `docs/{DOC}.md`

## Quick start

1. Get the source.
2. Isolate and install dependencies: {commands}.
3. Create local config from the example (placeholders only; real config stays ignored).
4. Run: `{RUN_COMMAND}`

## Configuration

**Environments:** {dev / test / prod / target}; how the project detects which; resources per environment. Production details as placeholders (`<PROD_ROOT>`, `${DATA_DIR}`).

> **Warning:** {resources never to access without explicit confirmation, and why}

| Setting | Description (+ why, if non-obvious) | Required | Default |
|---------|-------------------------------------|----------|---------|
| `{NAME}` | {…} | Yes/No | {…} |

## Usage

{Commands / endpoints / screens / options by audience. Keep in sync with in-product help.}

## Architecture

```
{project}/
├── {entry point}   # start-up
├── {module}/       # responsibility
├── docs/           # deep dives
├── plans/          # open plans (implemented/ = archive)
├── tests/
├── changelog.txt
└── backlog.txt
```

Principles: single source of truth · one layer per external interaction · assume dependencies fail · integrity before features · context-rich errors · constants, not magic numbers.

State and data: {owners, flow, retention}.

## Testing

`{TEST_COMMAND}` only. Default scope: {…}; slow/live tiers: {how}. See `docs/TESTING.md`.

## Contributing

Branch → follow `.github/copilot-instructions.md` → tests → docs + changelog → review.

## Troubleshooting

**{Symptom}:** {check 1} → {check 2} → {fix}

Logs: `{path}`. Known issues: `backlog.txt`. Reports include: errors, repro steps, expected vs actual, sanitised config.

## License / History

{license} · `changelog.txt`
