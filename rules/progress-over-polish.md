<!-- Progress Over Polish (L2)
Paste this whole file into the same instruction file as L1 (Discuss Before Diverging), right after it.
Optional layer: add it when you use a single model heavily (no subagent division of labor). -->

# Progress Over Polish (anchoring the main task)

> **What it governs**: after work starts. Prevents rabbit-holing on details, runaway test-writing, and imagined bugs during execution.
> **When it takes effect**: immediately after L1's alignment is released.
> **When it exits**: every ledger item meets its bar and the polish phase begins.

## 1. Build the ledger before starting

- Before acting, split the main task into 3–5 items, each with a one-sentence **verifiable completion bar**.
- **Never take any action that is not in the ledger**.
- At the start, state the task type (one line): `bug-fix` / `feature` / `refactor` / `explore`.

## 2. Action annotation

- Before every tool call, add a 1-line annotation: `serves ledger item N → expected output`.
- **Any action you cannot annotate = a drift signal**: stop immediately and return to the ledger.

## 3. Detail budget (adaptive, two layers)

- **Baseline** (tendency only; the two layers below decide the budget, and `explore` is not bound by them): `bug-fix` allows deeper detail; `feature` / `refactor` have the smallest detail budget; `explore` has no detail budget but must state its exploration goal in the annotation.
- **Blocking check (the main gate)**:
  - A detail that **blocks** the main path (cannot continue without it) → record it under the ledger item it blocks; may be relaxed up to the hard cap;
  - A detail that does **not block** ("found a risk", "feels like it should be tested") → **zero budget**: record 1 line and skip.
- **Rolling release**: completing 1 ledger item releases 1 batch of detail budget (≈3 tool calls); when the ledger is not advancing, the detail budget is 0.
- **Hard cap**: ≤ 20 tool calls per sub-problem, an unconditional circuit breaker that never takes part in adaptation. Changing approach, wording, or digging into causes at the same blocker does not reset the count.
- Adaptive layers are **deliberately kept at two** (blocking check + rolling release) — checkability over precision.

## 4. Circuit breaker on test timing

- Before the main path runs end to end, **do not** write unit tests / edge-case tests.
- Smoke-test / manually verify the main feature first, then add tests.
- Tests are a means of verification after completion, not a means of exploration.

## 5. Bug report threshold

- Before reporting any bug, give a reproducible path (input / steps / expected vs actual).
- What cannot be reproduced → label it a "risk assumption" and **record only, do not fix**.

## 6. Progress heartbeat

- Every 10 tool calls or upon completing a sub-step, restate in 1 sentence: `what the main task is / how many ledger items are done / whether it is currently drifting`.

## 7. Release

- Every ledger item meets its completion bar → enter the polish phase; this rule stops applying.

---

## Tunable constants

| Constant | Default | Notes |
|---|---|---|
| Reconnaissance file cap (L1) | 3 files | Index/routing/README exempt |
| Per-file read cap (L1) | 400 lines | Beyond it, read excerpts only |
| Ledger item count | 3–5 | Too few loses the anchor; too many becomes a burden |
| Detail batch | 3 tool calls | The unit of rolling release |
| Hard cap | 20 | Disaster prevention only; never adjust |
| Heartbeat interval | 10 calls | Too dense distracts; too sparse fails |
