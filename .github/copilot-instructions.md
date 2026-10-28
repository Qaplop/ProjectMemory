# Coding Playbook — {PROJECT_NAME}

Short, high-signal. `{BRACES}` = fill per project; delete rules that don't apply. Detail → `docs/`.

## Cardinal rules

Shape: DO / DON'T / WHY.

### 1) Single source of truth
- DO: one owner per piece of state/config; access only through it.
- DON'T: shadow copies.
- WHY: duplicates diverge; bugs surface far from the cause.

### 2) Search before creating
- DO: search for an existing implementation first; extend a close one.
- DON'T: near-duplicates.
- WHY: they drift apart. Easier code maintenance and fewer bugs.

### 3) Production safety
- DO: confirm the environment (dev/test/prod/target) before any file, data, network, or device operation.
- DO: get explicit user confirmation before touching any production resource — **every access, never standing**, and regardless of how "safe" the method is.
- DON'T: hardcode machine-specific paths, hosts, addresses.
- WHY: production mistakes are hard to undo and compromise production integrity.

### 4) Prefer copies of live resources
- DO: use a copy/export/snapshot instead of a live shared resource.
- DON'T: trust "read-only intent" — access itself may lock, write metadata, or disturb the owner. Judge by what happens to the live resource.
- DO: if direct access is confirmed (Rule 3), keep it short and narrow.
- WHY: incoherent concurrent access can corrupt, and surface later.

### 5) One layer per external interaction
- DO: route storage, network, hardware, files, OS calls through one layer owning retries, limits, caching, error translation.
- DON'T: raw external calls in business logic (one-off scripts excepted, labelled).
- WHY: one place for limits and failure handling.

### 6) Re-runnable operations, named access
- DO: make setup/migration/repair/deploy steps idempotent; order dependent steps after their prerequisites; test against the previous release's state.
- DO: change all mirrored copies of a structure together and keep a drift check.
- DO: address structured data by name/key, never position. Guard with a structural test (no positional access).
- DON'T: wildcard copy/merge between structures that should match; name every field.
- WHY: failures are silent wrong data, possibly for the data's whole lifetime.

### 7) Safe against repetition and concurrency
- DO: make every side effect (write, send, actuate, delete) safe if triggered twice, concurrently, or retried.
- DO: set the "in progress" marker first, before anything can interleave; stateless handlers re-check persisted state.
- DO: serialize per key; de-duplicate before the final write; await a step before follow-ups on the same resource.
- DO: put guards in shared helpers; keep a test listing all guarded entry points.
- DON'T: drop repeat triggers that carry new payload — queue/merge.
- WHY: double-clicks, retries, overlaps, re-entry.

### 8) Nothing acts before state is ready
- DO: gate external entry points on an "initialized" flag.
- DO: state owner refuses writes until its own load succeeded; reads never fabricate empty records that get persisted.
- DO: warn when a write replaces existing data with empty.
- DO: finish cleanup before an unconditional exit.
- WHY: an early trigger can erase real data.

### 9) Absence needs confirmation
- DO: treat only a confirmed "not found" as gone. Cache miss, permission error, timeout, rate limit, "not ready" = inconclusive: change nothing, log, retry.
- DO: create a replacement only after the old one is confirmed removed.
- DO: distinguish "not loaded", "absent", "unknown".
- WHY: cleanup on a transient gap destroys valid data.

### 10) Docs are part of the change
- DO: update affected docs/help/README/privacy statements in the same change; add a doc for significant features.
- DON'T: leave references to moved/renamed things.
- DO: reference docs as plain `docs/FILE.md § Section`, not markdown links.
- WHY: a stale doc misleads worse than none.

### 11) Plans live in the repo
- DO: save each approved plan to `plans/{ticket}-slug.md` as the first action, every time.
- DO: move it to `plans/implemented/` in the turn it's fully done (code, tests, commit).
- DO: write ≥1 manual test case per work item before marking it implemented.
- DO: put user-run one-off scripts in the project root; delete after use; never commit.
- WHY: plans in tool-private locations are lost.

### 12) No secrets or infrastructure details in the repo
- DO: use placeholders (`<PROD_ROOT>`, `${DATA_DIR}`, `HOST-NAME`); keep secret files ignored.
- DON'T: write real hosts, IPs, network paths, vendor/hardware names, or credentials anywhere tracked (docs, comments, commits, fixtures).
- DON'T: leave a secret file open in an editor with an active assistant — it may auto-attach and bypass ignore rules.
- WHY: the repo may go public.

### 13) Mask credentials at the type
- DO: give secret-holding objects a masked printed/debug form: exclude fields from the auto-representation **and** define an explicit one showing only `<set>`/`<empty>`.
- DO: mask to a constant — no length, prefix, suffix.
- DO: keep the secret-field list beside the definition; pin it with a test derived from it.
- WHY: traces, test failures, and formatting print every field.

### 14) Identifiable running artifact
- DO: monotonic build counter, +1 per batch of shipped-code changes; not for docs/tests/changelog; never renumber.
- DO: owner manages the release version.
- DO: expose version, build, and source fingerprint at runtime (log, status, device report). Same build + different fingerprint → trust the fingerprint.
- DO: check the build before analysing any log/report.
- WHY: file-copy/flash deployment makes "is it committed?" meaningless.

### 15) Keep time-critical paths responsive
- DO: keep event loops, UI/request handlers, control loops, interrupts short; move slow work out or slice it.
- DO: prefer one bulk operation over per-item loops; scrutinize start-up/migration code too.
- DON'T: assume a worker or timeout frees you from one long uninterruptible call — a timeout stops waiting, not the work.
- WHY: one stalled path stalls everything, usually misdiagnosed as slow network/hardware.

### 16) Memory files stay short
- DO: every project memory file (playbook, docs, cookbook, README, CLAUDE.md, changelog) is short, crisp, clear — one idea per line, no filler, repetition, or narrative.
- DO: tighten existing text before adding; delete what doesn't earn its place.
- DON'T: restate a rule in another file — reference by path.
- WHY: every token is read every session; short is faster for humans too.

---

## Quick reference (per project)

- {Common task} → `{helper}`
- Logging → `{logger}`, right level; no ad-hoc prints
- Loops over per-entity config → skip entities that no longer exist; keep their config for a retention period; "presence unknown" ≠ absent
- Automation acting on someone's behalf → declare needed permissions in one place, check at every enabling entry point; warn, never block
- Repeating operator-only failure → throttle the log line (per interval per target); name entity, target, missing capability

## Workflow guardrails

- Minimal diffs; reuse patterns.
- Preserve unknown fields when rebuilding structured state.
- Heavy one-off production data fix: `generate` (read-only on a copy → plain file) then `apply` (on the data machine; idempotent, explicit fields, unique-key writes, dry run default). Test on a non-prod copy; delete script and file after.
- Never switch the live working directory's branch to inspect another; use diff between refs, `show ref:path`, or a worktree.
- Multi-piece deploys (client+server, firmware+tool) = one feature: check every piece for changes. Non-prod needs no confirmation; prod always does, per piece.

## Common pitfalls (details: `docs/COPILOT_PITFALLS_COOKBOOK.md`)

1) Duplicate helpers → search (Rule 2).
2) Context (identity, scope, flags, config) not threaded through calls.
3) Fields lost when rebuilding state.
4) Positional access (Rule 6).
5) New field empty until next refresh → backfill.
6) Cleanup on inconclusive signal (Rule 9).
7) Delete-then-unconditionally-create.
8) Truthiness fallback chains (`a or b`) → explicit presence check.
9) Placeholder defaults ("Unknown") look real.
10) Overloaded fields / two facts in one field.
11) Concurrent detect-then-persist (Rule 7).
12) Fire-and-forget then foreground step on same resource.
13) Single-entity action calling a "process everything" helper.
14) Time-limited tokens/links → use immediately.
15) Start-up code that runs twice (unit loaded under two names) → guard by name.
16) Blanket async mocks make sync methods async → fix the shared fixture.
17) Sentinels (`-1` = unknown) in numeric data.
18) Tests covering only one code path → stratify.

## Implementation workflow

**Before:** search (Rule 2); read module docs; trace data flow; check environment (Rule 3); plan rollback; fold matching deferred backlog items into the change and remove them.
**While:** follow the rules and quality standards; log at the right level.
**After:** verify it solves the problem; check dependents, paired acquire/release, error paths; update docs (Rule 10); bump build (Rule 14); add changelog; run `{TEST_COMMAND}` — the single canonical command, never a hand-built variant (extra filters as trailing args).

## Code quality

- Explicit types where supported; document params, returns, units, failures.
- Don't remove or rewrite existing comments unless asked; outdated → prefix "OUTDATED", add new below.
- A comment qualifying a branch ("usually X") → the condition must test X.
- Catch specific expected failures; separate "confirmed absent" from "could not determine".
- Use the shared retry helper for flaky/rate-limited calls; log with context (who, what, target); user-facing errors understandable.
- Unavailable-by-design states (maintenance, start-up) → distinct condition, retried, not logged as failure.
- A check/alert is evidence only if it can fire — prove reachability before trusting silence.
- Samples: stratify by code path. Validate estimators against an independent measurement.
- Profiler hotspot = suspect; confirm with matched before/after; record changes that did nothing.

## Changelog

Update `changelog.txt` for every code change, unasked.
- Newest entry at the top, below the header.
- Heading `YYYY-MM-DD (N)`, today's date; N restarts at 1 each date; never renumber. Don't re-derive this from the file.
- A few lines: Fixed/Added/Changed + why, files, `Build N.`, whole-suite test total verbatim from `{TEST_COMMAND}`.
- Roadmap is separate: `backlog.txt`.

```
2026-01-15 (2)
- Fixed: {symptom}. {cause/fix}. File: {path}. Build 123. {N} tests pass.
```

## Backlog

Maintain `backlog.txt` (root) unasked, like the changelog.
- Add: ideas, known issues, and deferred fixes you find but don't do now.
- Remove: an item the moment it's done (record it in the changelog).
- One line per item, tagged `[idea]`, `[bug]` or `[next-touch]` (= do when that code is next edited anyway).

## Docs index (one line each: `docs/FILE.md: scope`)

- `docs/ARCHITECTURE.md`: modules, data flows, decisions
- `docs/INTEGRATIONS.md`: external systems, limits, failure modes
- `docs/TESTING.md`: tiers, fixtures, coverage, CI
- `docs/PERFORMANCE.md`: measurements, what worked / didn't
- `docs/COPILOT_PITFALLS_COOKBOOK.md`: recurring mistakes
- `README.md`, `backlog.txt`, `changelog.txt`

When in doubt: search the code, then the docs, then ask.
