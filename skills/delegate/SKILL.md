---
name: delegate
description: Orchestrate work by handing execution (test runs, browser QA, eval rounds, data gathering, mechanical edits) to a smaller, cheaper model — by default Sonnet at low effort — while you, the higher model, keep every judgement: the plan, the expected answers, the grading, the fixes and the final verdict. Before starting, make sure the feature or fix has a PHASES.md of phases → subtasks (create it if missing), and commit and tick each subtask as it lands. Use this whenever the user says "delegate", "hand this to sonnet/haiku", "run it through a smaller model", "low effort agent", "you only monitor", "have a subagent run the tests/evals", or asks for repeated test/eval rounds you should supervise rather than do by hand.
---

# Delegate

You are the **orchestrator**. A smaller model is the **worker**. The worker does the legwork; you
decide what "correct" means, check its evidence, fix what it finds, and tell the user the truth.

The point of delegating is cost and speed, never lower quality. A cheap run that you accept without
checking is worse than no run, because it produces confident wrong conclusions. So the rule of thumb
is: **the worker gathers, you judge.**

## 1. Before anything runs: PHASES.md

Every delegated feature or fix is tracked in a PHASES.md, so progress survives context resets and
each step is reviewable.

1. Look for an existing tracker: `docs/PHASES.md`, `PHASES.md`, or `phases/*.md`. Follow the
   project's own convention and its doc rules (some repos cap the number of docs, so add a section
   instead of a new file).
2. If none covers **this** feature or fix, create one before starting work (copy
   `references/phases-template.md`). If a PHASES.md exists for a different feature, add a new
   top-level section for this one rather than mixing subtasks.
3. Structure: goal → decisions → phases → numbered subtasks (`1.1`, `1.2` …) with checkboxes. Each
   subtask is small enough to be **one commit**.
4. Subtask 0.1 is the plan itself. Commit it first.
5. When a subtask lands, tick its box and record the commit id
   (`- [x] **2.3** Cap reply length at sentence boundaries. — F494 (621f3f3)`). Commit messages
   follow the project's style (e.g. `F<NNN>: summary`) and the user's attribution rules.

Keep the plan honest: if a round of testing reveals new work, add subtasks for it; if a subtask turns
out to be wrong, mark it dropped with one line saying why. Don't silently rewrite history.

## 2. What to delegate, and what never to

Delegate:
- Running test suites, builds, lint, and reporting results.
- Browser QA and eval rounds: sending prompts, measuring timings, taking screenshots, recording raw
  outputs.
- Gathering ground-truth data (API reads, DB queries) into a file.
- Mechanical, fully specified edits (rename, update a fixture to a value you computed).

Keep for yourself:
- Scoping the task, writing PHASES.md, choosing the rubric and the pass bar.
- Deciding the **expected** answer for each test case, or at least checking the worker's expected
  answers against the ground truth before trusting any grade.
- Root-causing and fixing bugs. The worker reports symptoms; you trace the cause through the code.
- Code on money, security, auth, or data-loss paths. If a worker touched it, review the diff line by
  line.
- The final verdict and what you tell the user.

## 3. Launching the worker

Use the Agent tool with `model` set to the smaller model and `effort` set to the level the user
asked for (default: `model: "sonnet"`, `effort: "low"`). Run it in the background so you can keep
working, for example by fixing code while the eval runs.

The worker starts cold and sees none of this conversation, so the brief must stand on its own.
Include:

- **Goal**, in one paragraph: what is being tested and why.
- **Hard limits**: e.g. "do not edit source, write docs or commit; scratch files only in
  `/tmp/qaN/`".
- **Exact setup**: paths, ports, how to start or restart services (call out anything that doesn't
  hot-reload), which browser or tool to use, and how to authenticate.
- **Ground truth first**: what data to capture before testing, and where to save it.
- **The cases**: a numbered list, reused across rounds so results stay comparable. Add new cases
  rather than replacing old ones.
- **The rubric**: precise scoring (e.g. correct 0/1/2 against ground truth, chart matches text,
  word count with a stated counting rule, latency measured a stated way).
- **Previous round's numbers and failure list**, so the worker reports each one as fixed or still
  broken, with quotes.
- **Stop conditions**: "if X errors, read the log, report and stop; don't keep sending."
- **Report format**: a table, then totals against the previous round, then a list of every failure
  with evidence (quotes, file paths).

For each new round, write a fresh brief that carries forward the prior findings. Don't rely on the
worker remembering anything.

## 4. Monitoring

- Don't poll or tail the worker's transcript; you're notified when it finishes. Use the time for
  orchestrator work.
- Never report or predict results before the notification arrives. If the user asks, say it's
  still running.
- If the user hasn't heard from you during a long stretch, give a one-line status of what you're
  doing.

## 5. Verifying the worker's report (this is where quality is kept)

A smaller model at low effort will sometimes mis-measure, mis-count or over-grade. Before you act
on a report:

- **Spot-check evidence yourself.** Open a few screenshots, especially the failures and anything
  surprising. Re-run one key command if a claim hinges on it.
- **Look for measurement artifacts.** Check whether a timing captured the user's own message
  appearing rather than the reply, or whether a word count included lines the rubric excludes.
  Correct the comparison and say so.
- **Re-judge borderline grades** against the ground truth with your own reasoning. Your grade wins.
- **Separate symptoms from causes.** "Chart shows TRAVEL $500" is a symptom. Find the code path
  before deciding what to change.
- **Don't treat a regression as noise.** If a score dropped, find out whether your change caused it
  before running another round.

## 6. Fix → re-run loop

1. Fix the root causes the round exposed, each as its own subtask and commit, with a test that
   fails if the fix breaks.
2. Run the project's checks yourself, or delegate them and verify the output.
3. Launch the next round with the same cases, the new numbers to beat, and the open-failure list.
4. Stop when the targets are met, or when another round would mostly measure model variance rather
   than your changes. In that case, tell the user which lever is left: a different model, a new
   tool, or a code-level guard.

Prompt-only fixes plateau. When the model keeps guessing, check whether it simply lacks the data,
and give it a tool or a precomputed field rather than another prompt rule.

## 7. Reporting to the user

- Lead with the verdict: did it work, and what's still broken.
- Give a round-over-round table (correctness, quality, length, latency), noting any measurement
  caveats.
- List what you confirmed fixed and what remains, with the concrete example for each.
- State what was committed and pushed (push only when asked; ask whether to open a PR or push
  directly).
- Don't pass along the worker's prose wholesale. Say what you verified, and flag anything you
  couldn't verify.
