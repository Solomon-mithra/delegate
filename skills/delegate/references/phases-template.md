# <Feature or fix> — phases

Branch: `<branch>` (from `<base>` at `<sha>`). Each subtask is one commit, `<commit style, e.g.
F<NNN>: summary>`, following the project's attribution rules. Tick the box and record the commit when
it lands.

## Goal

<One paragraph: what the user should be able to do when this is finished, and what "done" means,
including the measurable bar, e.g. "eval correctness ≥ 42/44, replies ≤ 40 words, p50 ≤ 3 s".>

## Decisions

- <Each design choice and why. Note what was rejected, so a later reader doesn't re-litigate it.>

## Delegation

- Worker: <model + effort, e.g. Sonnet, low effort>. Orchestrator keeps the plan, the expected answers,
  grading, fixes and the final verdict.
- Eval set: <where the cases live, e.g. `/tmp/qa/qs.json`; how ground truth is captured>.
- Rubric: <scoring rules, including how words and latency are counted>.

## Phase 0 — plan

- [ ] **0.1** This document. — <commit>

## Phase 1 — <baseline / first slice>

- [ ] **1.1** <subtask: one commit's worth of change, plus the check that proves it>. — <commit>
- [ ] **1.2** …

## Phase 2 — <next slice>

- [ ] **2.1** …

## Rounds

| Round | Change under test | Correctness | Quality | Length | Latency p50 / max | Notes |
|---|---|---|---|---|---|---|
| 1 | baseline | | | | | |

## Open issues

- <Found by a round but not yet scheduled as a subtask.>
