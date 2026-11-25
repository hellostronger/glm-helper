<!-- Discuss Before Diverging (L1)
Paste this whole file at the TOP of your instruction file (AGENTS.md / CLAUDE.md / global rules), before project-specific rules.
Works across ZCode / Claude Code / Codex / Cursor / Antigravity. -->

# Discuss Before Diverging

> **What it governs**: before work starts. When the requirement has ≥ 2 reasonable interpretations in product intent, scope of change, or architectural approach, align first, then act.
> **When it exits**: alignment complete; during execution, *Progress Over Polish* takes over.

## Trigger signals (any one fires)

- The change spans ≥ 2 modules or directories;
- Product behavior has ≥ 2 user-perceivable variants;
- The requirement contains unbounded verbs ("optimize", "refactor", "clean up", "support X a bit");
- Any branch would change public APIs, data structures, or the execution mode;
- Fallback: ≥ 2 plausible product / scope / architecture interpretations exist.

## 1. Alignment-turn output contract (binds visible artifacts, not internal reasoning)

- The check-in turn **presents forks and options only**: 2–4 concrete options, or one open question — no implementation design, edge cases, or refactors for any branch.
- **The sole checkpoint is the option count**: outside 2–4 is a violation, countable after the fact.
- Internal reasoning has **no sentence quota**: this section governs what appears in the check-in turn, not how much gets considered.
- Implementation details belong to the post-alignment single execution path and must not leak into the check-in.

## 2. Scouting cap (at most 1 read-only pass before asking)

- Counted in **files + lines**, not reads: ≤ 3 files, ≤ 400 lines per file; OR 1 batch of concurrent read-only subagents (`research` / `Explore`).
- Subagents are **bound by the same quota**; delegation is not a bypass. The whole batch shares one cap.
- Files over 400 lines are **read in slices**: locate with `grep` / `Glob` first, then take the relevant range by offset/limit.
- **Index / routing / README-type files are exempt** from the cap.
- Full scouting is **the exception, not the default**: allowed for unambiguous tasks, but must be declared as "full scouting" in the action tag.

## 3. Zero unrequested scope

- Never invent unmentioned future requirements; never bundle "while we're here" changes.
- Adjacent issues → log **1 line each**, hand the decision back to the user, **do not fix this round**.

## 4. Single-turn check-in

Takes precedence over multi-turn one-question-per-turn / no-menu rules of brainstorming skills (unless the user explicitly asks for open brainstorming or an interactive design interview).

- Sync known codebase facts in 2–3 sentences and resolve the ambiguity **in the same turn**, merging any model-tier / execution-mode recommendation.
- **Finite choices** (architecture trade-offs, scope boundary, execution mode): call the structured question tool (`ask_question` / `AskUserQuestion` / `AskQuestion`) with 2–4 concrete options, `(Recommended)` first. No such tool → numbered plain-text list, `1)` recommended.
- **Open intent**: ask exactly 1 concise plain-text question, no steering options.

## 5. Bypass & release

- **Bypass**: fully disabled when the user explicitly asks for open brainstorming, side-by-side comparison, or an interactive design interview (e.g., `/grill-me`).
- **Release**: the user's answer lifts the circuit breaker.
- **Re-trigger**: still ≥ 2 readings after the answer → at most ONE follow-up; then proceed under the stated assumption, flagged explicitly in the reply.
