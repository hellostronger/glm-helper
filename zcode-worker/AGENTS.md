---
description: Full anti-drift ruleset — pre-work alignment + in-execution anchoring + model division of labor
alwaysApply: true
---

# Rule 1: Discuss Before Diverging (align before forking)

> **What it governs**: before work starts. When the requirement has ≥ 2 reasonable interpretations in product intent, scope of change, or architectural approach, align first, then act.
> **When it exits**: it exits once the main-task ledger is built and alignment is complete; during execution, Rule 2 takes over.

## Trigger signals (any one match)

- The change spans ≥ 2 modules or directories;
- Product behavior has ≥ 2 user-perceivable variants;
- The requirement contains unbounded verbs ("optimize", "refactor", "clean up", "support X a bit");
- Any branch would change public APIs, data structures, or the execution mode;
- Fallback: ≥ 2 reasonable product / scope / architecture interpretations exist.

## 1. Alignment-turn output contract (binds visible output only, not internal reasoning)

- Aside from the fact sync in section 4, the alignment reply **presents only the fork and the options**: 2–4 concrete options, or 1 open-ended question. Write no implementation design, edge cases, or refactoring plan for any branch.
- **The sole checkpoint is the option count**: anything outside 2–4 is a violation; it is countable and verifiable after the fact.
- Internal reasoning has **no sentence quota**: this section governs "what appears in the alignment turn", not "how much the model considered".
- Implementation details belong to the single execution path **after alignment** and must not leak into the alignment turn early.

## 2. At most 1 round of read-only reconnaissance before alignment

- The unit of count is **files + lines**, not "how many times it read": ≤ 3 files, ≤ 400 lines per file; or 1 batch of concurrent read-only subagents (`research` / `Explore`).
- Subagents are **bound by the same quota**; "delegating" must not be used to bypass the file / line caps. The whole batch of subagents shares one cap, which does not scale with the number of agents.
- For files over 400 lines, **read excerpts only**: locate with `grep` / `Glob` first, then take the relevant passages by offset/limit.
- **Index / routing / README-type files do not count toward the cap**.
- Full reconnaissance is **the exception, not the default**: allowed when the requirement is clear, but "full reconnaissance" must be declared in the action annotation.

## 3. No incidental expansion

- Never invent unmentioned future requirements, and never bundle in "while we're at it" changes.
- Record each peripheral issue you find in **1 line each**, hand the decision back to the user, and **do not fix it this round**.

## 4. Single-turn consolidated confirmation

This takes precedence over the one-question-per-turn / no-menus rules of `superpowers:brainstorming` (unless the user explicitly asks for open-ended brainstorming or an interactive design interview).

- First sync the known code facts in 2–3 sentences, and complete the alignment **in the same turn** (merge in model-tier / execution-mode recommendations as well).
- **Choosing among finite options** (architecture trade-offs, scope boundaries, execution mode): call the structured question tool (`ask_question` / `AskUserQuestion` / `AskQuestion`) to give 2–4 concrete options, with `(Recommended)` on top. **Without such a tool, fall back** to a numbered plain-text list where `1)` is the recommended item.
- **Open-ended goal clarification**: ask exactly 1 concise plain-text question, with no leading options.

## 5. Bypass and release

- **Bypass**: when the user explicitly asks for open-ended brainstorming, side-by-side comparison of options, or an interactive design interview (such as `/grill-me`), this rule **stops applying entirely**.
- **Release**: the user's answer releases the circuit breaker.
- **Re-trigger**: if the answer still has ≥ 2 interpretations, **at most one** follow-up confirmation is allowed; if it is still ambiguous after that, proceed on a declared assumption and state that assumption explicitly in the reply.

---

# Rule 2: Progress Over Polish (anchoring the main task)

> **What it governs**: after work starts. It prevents rabbit-holing on details, runaway test-writing, and imagined bugs during execution.
> **When it takes effect**: immediately after Rule 1 is released by alignment.
> **When it exits**: it stops applying once every item in the main-task ledger meets its bar and the polish phase begins.

## 1. Build the ledger before starting

- Before acting, split the main task into 3–5 items, each with a one-sentence **verifiable completion bar**.
- **Never take any action that is not in the ledger**.
- At the start, state the task type (one line): `bug-fix` / `feature` / `refactor` / `explore`.

## 2. Action annotation

- Before every tool call, add a 1-line annotation: `serves ledger item N → expected output`.
- **Any action you cannot annotate = a drift signal**: stop immediately and return to the ledger.

## 3. Detail budget (adaptive, two layers)

- **Baseline** (it expresses a tendency only and grants no budget; the two layers decide the budget, and `explore` is not bound by them): `bug-fix` allows deeper detail; `feature` / `refactor` have the smallest detail budget; `explore` has no detail budget but must state its exploration goal in the annotation.
- **Blocking check (the main gate)**:
  - A detail that **blocks** the main path (cannot continue without it) → record it under the ledger item it blocks; it may be relaxed up to the hard cap;
  - A detail that does **not block** ("found a risk", "feels like it should be tested") → **zero budget**: record 1 line and skip it.
- **Rolling release**: completing 1 ledger item releases 1 batch of detail budget (3 tool calls); when the ledger is not advancing, the detail budget is 0.
- **Hard cap**: ≤ 20 tool calls per sub-problem, an unconditional circuit breaker that does not take part in adaptation. Changing approach, changing wording, or digging into causes at the same blocker does not reset the count.
- The number of adaptive layers is **deliberately kept at two** (blocking check + rolling release), with no further conditions stacked on — checkability over precision.

## 4. Circuit breaker on test timing

- Before the main path runs end to end, **do not** write unit tests / edge-case tests.
- Smoke-test / manually verify the main feature first, then add tests.
- Tests are a means of verification after completion, not a means of exploration.

## 5. Bug report threshold

- Before reporting any bug, you must give a reproducible path (input / steps / expected vs actual).
- What cannot be reproduced → label it a "risk assumption" and **record only, do not fix**.

## 6. Progress heartbeat

- Every 10 tool calls or upon completing a sub-step, restate in 1 sentence: `what the main task is / how many ledger items are done / whether it is currently drifting`.

## 7. Release

- Every ledger item meets its completion bar → enter the polish phase, and this rule stops applying.

---

# Tunable constants

| Constant | Default | Rule | Notes |
|---|---|---|---|
| Alignment option count | 2–4 | Rule 1 | Anything outside is a violation; countable and verifiable |
| Reconnaissance file cap | 3 | Rule 1 | Index/routing/README files are exempt |
| Per-file read cap | 400 lines | Rule 1 | Beyond it, read excerpts only |
| Ledger item count | 3–5 | Rule 2 | Too few loses the anchor; too many becomes a burden |
| Detail batch | 3 tool calls | Rule 2 | The unit of rolling release |
| Hard cap | 20 | Rule 2 | Disaster prevention only; never adjust |
| Heartbeat interval | 10 calls | Rule 2 | Too dense distracts; too sparse fails |

---

# Rule 3: Division of labor between models

- The main model is responsible for: alignment, analysis, task decomposition, dispatching work, and review.
- Any implementation work already split into a single acceptable unit (writing code, editing files, running commands, adding tests) goes to the `worker` subagent; the main model does not write it by hand.
- When dispatching work, all of these must be provided: goal, files involved, acceptance criteria, prohibitions.
- After `worker` returns, the main model must personally read the changes before reviewing; it must not rely on the report alone. If the result is unacceptable, dispatch once more with the specific problems.
- The main model running verification commands itself during review counts as a violation: put the commands needed for verification into the work order, and have `worker` run them and return the raw output.
- When no `worker` is available in the current session (not loaded, or disabled), or when `worker` cannot do it (reports that it cannot complete, or is still unacceptable after one more dispatch), the main model steps in and does that implementation work itself.

## Delegability test

- **Sole criterion**: if you can write a one-sentence verifiable completion bar and list the files involved → it can be delegated; if you cannot → it is still at the decision stage, so leave it to the main model. Every ledger item should itself be a work order.

| Step that can go to `worker` | Precondition |
|---|---|
| Adding code against a settled interface (functions, modules, files) | The main model settles the signature, inputs/outputs, and error handling first |
| Mechanical edits: renames, bulk replacements, template-generated repetitive code | The rules can be stated in a few sentences |
| Running builds, tests, lint, and scripts, returning raw output | Output only, no conclusions |
| Adding unit tests for existing functions | The main path already runs; the main model provides the test-case list |
| Fixing a well-located bug | The repro path and the fix are confirmed; `worker` only implements it |
| Keeping comments and docs in sync with code | The main model provides the content |
