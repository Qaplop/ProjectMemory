# Pitfalls Cookbook

Recurring mistakes: symptom → cause → fix. Referenced by the playbook. Plain-text doc refs only.
Add entries numbered, titled by *mechanism*, template at the bottom. Pseudocode only.

## A. State and data

1. **Positional access to evolving structures** — Symptom: value holds another field's content, silently. Cause: index-based access; structure later changes. Fix: access by name; structural test that positional access is absent (a behavioural test can't fail before the breaking edit).
2. **Wildcard copy/merge between "matching" structures** — Symptom: values land in wrong fields, no error. Cause: matched by position; layouts differ after different histories. Fix: name every field on both sides (reads and writes); parity check; verify real layout, not source text.
3. **Step placed before its prerequisite** — Symptom: fails only on existing installs. Cause: "create if missing" is a no-op there. Fix: dependent step right after its provider; test from the previous release's state.
4. **Lock/transaction/handle left open after a no-op** — Symptom: others intermittently "busy"; periodic job breaks when it had nothing to do. Cause: release conditional on "something changed". Fix: release unconditionally, incl. error paths.
5. **New field empty until next refresh** — Fix: backfill on introduction or register it with the force-refresh mechanism; skip fields whose default is already right.
6. **Overloaded field (several kinds of value)** — Symptom: bogus placeholders; junk kept alive. Fix: check for a real constraint before integrity helpers; filter every "is referenced" check by the discriminator; one registry of referencing places, tested.
7. **Two facts in one field** — Symptom: writing A destroys B. Fix: split into independent fields with separate write paths. Tell: comment "no separate X needed, Y means both".
8. **Stored marker used to decide "mine to modify"** — Cause: several writers stamp it under different conditions. Fix: re-derive from current authoritative state.
9. **Sentinels in numeric data (`-1` = unknown)** — Cause: sums treat it as a number. Fix: guard every aggregate, render "unknown", derive nothing; test for unguarded aggregates.
10. **Snapshot treated as live** — Fix: re-resolve on read or document/test snapshot semantics. "Last saved" ≠ "created".
11. **Persistent reference outlives its target** — Fix: keep targets as long as references work; handle "missing".
12. **Deleting what will be needed again** — Fix: retention period before purge; every new per-entity store covered by the purge.
13. **Coverage check inspects one of several sources** — Fix: expand every source (items, groups, guests) in every check; search each accessor after changes.
14. **Time-limited tokens/links** — Fix: consume on receipt (background task), await later; never re-touch the link. Failure appears weeks later, looking like flaky infrastructure.

## B. Concurrency and timing

15. **Blocking a time-critical path** — Symptom: system freezes; log gap then burst of identical elapsed times. Fix: respond first, move/slice slow work; yield in big loops.
16. **Offloading can't help one uninterruptible call** — Fix: shrink the call's scope; interruptible loops can be sliced or moved.
17. **Offloaded ≠ fast** — Fix: one bulk operation beats a per-item loop (4× in one case); scrutinize start-up/migration code ("runs once" ≠ fast).
18. **Waiting per item in a data-sized loop** — Fix: batch, bounded concurrency, or worker.
19. **Timeout cancels the wait, not the work** — Fix: cooperative cancellation/limit in the work; orphaned work must not corrupt state.
20. **Automatic memory-management pauses** — Symptom: multi-second freeze, no log output, then identical elapsed times. Fix: log pauses above a threshold first; shrink what's walked (mark long-lived start-up data permanent, refresh in maintenance windows); re-check tuning after runtime upgrades.
21. **Cycles in third-party object graphs** — Fix: break them when done, only after proving you own the object (a shared cache may serve the same reference).
22. **Exit from inside a sub-task** — Symptom: hang. Cause: only the sub-task ends. Fix: clean up first, then terminate the whole process; unconditional exit skips exit handlers.
23. **Racing detect-then-persist callers** — Symptom: integrity violation recurring for hours. Cause: both add to a shared in-memory collection; failed write never repairs it. Fix: serialize per key; de-duplicate before the write.
24. **Fire-and-forget then foreground action on same resource** — Symptom: toggle doesn't show until refresh. Fix: await; guard rapid repeats.
25. **Repeated trigger mid-flow** — Symptom: duplicate effects, even client crash. Fix: "in progress" marker first, before any suspension; disable input while busy; audit *every* entry point editing that surface. Drop only if the repeat carries no new payload; else queue/merge.
26. **Start-up code runs twice** — Symptom: callbacks fire twice, near-identical log pairs. Cause: entry script also imported by name → executed again; shared-collection mutations double. Fix: guard by name, not identity; better, break the import cycle.
27. **Entry points live before state loads** — Symptom: "you have no data" right after restart; real data deleted. Cause: read path writes a fabricated empty record. Fix (all three): initialized-flag gate; owner refuses writes until loaded; warn on write-replaces-with-empty.
28. **Maintenance mode** — Fix: every access path (primary, fallback, pooled, API) checks it and raises a specific "unavailable by design" error; background jobs retry quietly; endpoints answer "maintenance".

## C. Logic and design

29. **Truthiness fallback (`a or b`)** — Symptom: "none" reverts to old value. Fix: explicit presence check.
30. **Placeholder defaults** — `get_name(id)` returns truthy "Unknown", so `or id` never applies. Fix: pass the fallback you want, or a null marker.
31. **Comment says "usually X", condition doesn't test X** — Fix: encode X; test the "not X" case.
32. **Two on-ramps, one safety check** — Fix: enumerate every path creating the state; check at the lowest shared point too.
33. **Free-form input misclassified** — Fix: take the identifier reading only with an unambiguous marker (e.g. leading symbol); else try name first.
34. **Repair for something start-up already regenerates** — Fix: check the normal start-up path first.
35. **Copied "block everything" flag** — Fix: comment what each blocking flag protects; prefer non-blocking.
36. **Single-entity action → "process everything" helper** — Fix: add an optional scope parameter; read helper docs before wiring.
37. **Local cache miss ≠ remote deletion** — Fix: purge only on confirmed not-found; other errors leave state untouched.
38. **Delete-old then unconditional create-new** — Symptom: duplicates after transient failure. Fix: create only after delete confirmed; else skip this cycle.
39. **Repeating failures flood logs** — Fix: throttle per interval per target; name entity, target, missing capability.
40. **Several rule systems changing the same thing** — Symptom: flips every cycle. Fix: systems only vote; apply the combined result once; anything wanted is never removed.
41. **Private framework hooks** — Fix: never name your methods like them; prefix.
42. **Hook registered via decorator instead of assignment** — Symptom: guard never runs. Fix: assign directly; test it blocks.
43. **Client-side caches** — New feature looks broken. Fix: reload client, check server registration log before debugging.
44. **Stream/encoding differences across platforms** — Symptom: tool dies silently on one OS or mangles non-ASCII. Fix: blocking read in a worker; explicit encodings; reproduce by launching directly and sending a raw request.
45. **Hard limits fail totally** — One over-long label/payload fails the whole registration. Fix: length-validation test over all strings; truncate at construction; navigation entries count toward list limits.
46. **Renderers trim/reorder text** — Whitespace-only lines vanish → use a visible/zero-width separator. Right-to-left text scrambles the whole mixed line → direction marks around the name *and* at every following field transition; copy a proven format; test on real clients.
47. **Per-item directory scans** — Symptom: instability (even kernel-level) with real-time antivirus past a size threshold. Cause: items × files callbacks. Fix: scan once, map, look up; exclude data dirs where policy allows.

## D. Testing, measurement, verification

48. **Blanket async mock makes sync methods async** — Symptom: `if not r.is_done()` takes the wrong branch (truthy coroutine). Fix: real sync mock in the *shared fixture*, not per test.
49. **Estimator measures nothing** — Cause: walker misses objects using another attribute mechanism; per-entry `seen` double-charges shared parts. Fix: generic referent traversal; stop at types/modules/functions; seed `seen` with shared roots; validate against an allocation tracer; build fixture payloads per instance (literals get folded into one shared object).
50. **Gate that can't fire** — Symptom: "not enough data yet" forever. Fix: prove the observation is emittable (reachability, forced positive test).
51. **Sample proves nothing per path** — Fix: stratify by code path; report per stratum.
52. **Undisciplined performance work** — Fix: matched load; utilisation, not just latency; hotspot = suspect until before/after confirms; record what did nothing; log-gap analyser (gaps at *varying* places = global pause; *fixed* place = one call). Windows: force UTF-8 when piping logs.
53. **Recovery proves structure, not content** — Fix: verify content separately (counts vs known-good copy, spot checks).
54. **Logs from an unknown build** — Fix: playbook Rule 14; if build equal but fingerprint differs, say so.

## Entry template

```
N. **<Mechanism>** — Symptom: … Cause: … Fix: … (guard/test). Evidence: date, how confirmed. See: docs/FILE.md § Section
```
