---
name: mock-use-subagents-wisely
description: >-
  Turn on, for the rest of the session, a lean-orchestrator mode: this session
  plans and checks, and delegates real work to subagents on three model tiers
  (Top, Mid, Low) according to a spending level given as $ to $$$$$ (default
  $$$). Takes the level and an optional task as arguments; the user can change
  the level later. Works in Claude Code and in any other coding agent. MANUAL
  TRIGGER ONLY: apply when the user invokes /mock-use-subagents-wisely.
disable-model-invocation: true
---

# Use subagents wisely

A session mode. Once invoked it governs how this session carries out the task in the invoking prompt and every later prompt, until the user says to stop. It decides two things only: how much the orchestrator does itself, and which model tier each subagent runs on.

## Model tiers

The rules speak of three tiers, never of model names, so they survive renames and new versions. At invocation, map the tiers to the models this harness can actually give a subagent, and state the mapping in one line.

| Tier | Meaning | Example, Claude Code in late 2026 |
|---|---|---|
| **Top** | the most capable model offered, usually the one with its own tight usage limit | Fable |
| **Mid** | the everyday strong model | Opus |
| **Low** | the fast and cheap model | Sonnet |

How to map in any harness (Claude Code, Codex, Gemini CLI, or whatever runs this skill), on any day:

- Find what subagent models the harness offers: the subagent tool's model parameter and its allowed values, the harness's help output or settings, or the model picker. Prefer a short alias over a dated id, because aliases outlive versions.
- Rank the offered models by capability and cost, whoever makes them. The costliest and most capable is Top, the cheapest is Low, the one in between is Mid. With more than three, pick the three that fit those roles best and ignore the rest.
- Only two models offered: that harness has Top and Low, and Mid work goes to Top at `$$$$` and above, to Low below that.
- Only one model offered: delegate for a small orchestrator context anyway, and ignore the model column.
- The session's own model is irrelevant to the mapping. The rules are the same whether the orchestrator runs on Top or Mid.

## Arguments

`/mock-use-subagents-wisely [LEVEL] [task...]`

- **LEVEL** is a leading token made only of `$` characters, one to five of them. Everything after it is the task to start on.
- **No level given**: ask with the question below, then arm the fallback. Start the task once the level is known.
- **Changing level later**: a message that is only `$`-characters (for example `$$$$`), or a re-invocation with a new level, sets the new level for the rest of the session. Acknowledge in one line and carry on. The level is session-only; it is never saved to memory.

### Asking for the level

1. Run `date`. Add two minutes, round up to the next whole minute, and schedule a one-shot job for that minute (CronCreate in Claude Code; the harness's equivalent elsewhere) with the prompt: "If the user has not yet chosen a $ level for mock-use-subagents-wisely, use $$$ and start the task from the invoking prompt. If they have, do nothing." Remember the job id. A harness with no scheduler skips this step and waits for the answer.
2. Ask with the harness's question dialog (AskUserQuestion in Claude Code). Offer the five levels with one-line descriptions, `$$$` first and marked (Recommended).
3. When the user answers, delete the scheduled job, then start.
4. If the job fires first, the level is `$$$`.

## How the orchestrator works

- **Stay lean.** The orchestrator's job is to plan, delegate, check results and keep track. Subagents read the large files, write the code, run the tests and take the screenshots. This holds at every level and on every model, because the goal is a small orchestrator context, not a cheap one.
- **Trivial requests are answered directly.** A one-line question, a quick lookup, an edit of a couple of lines: do it yourself. Delegation is for real work.
- **Name the model on every subagent call.** Pick the tier from the level table below and pass its mapped model.
- **Parallel agents own disjoint files** and re-read a file before editing it.
- **One test suite at a time.**
- **One Top subagent at a time**, at every level.
- **Low agents** are told to finish in one go and not end their turn while a background command runs.
- **Final review.** When a task, or a batch of tasks, is done, one read-only review subagent reads the whole change for bugs, regressions, dead code and wording. Fix what it finds with Mid, or Top where the table allows. The reviewing tier comes from the table.

## Level table

Work not named in a row goes to the next tier down.

| Level | Top | Mid | Low |
|---|---|---|---|
| `$` | never | all packages, all bugs, the final review | easy work with a clear spec: wording, docs, screenshot runs, tests for a finished module, mechanical edits |
| `$$` | the final review, and a bug Mid failed to fix twice | everything else | as above |
| `$$$` (default) | the final review, hard bugs with an unknown cause, a very complex package, a messy fix that spans several files, a design question Mid got wrong once | every other package and ordinary bug, the read-only review after each wave | as above |
| `$$$$` | everything in `$$$`, plus every package that touches core state, rendering or long-running jobs, and the review after each wave | easy and ordinary tasks | never |
| `$$$$$` | the default for any non-trivial work | easy mechanical tasks only: wording, docs, screenshot runs, simple tests, renames | never |

`$$$$$` exists for the hours before a usage limit resets, when unused Top budget is lost. It is chosen by the user, never by the session.
