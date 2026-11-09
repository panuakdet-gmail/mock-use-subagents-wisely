# use-subagents-wisely

A skill for [Claude Code](https://claude.com/claude-code) that keeps your main session small and hands the real work to helper agents, each on the model that fits the job. You pick how much to spend with a level from `$` to `$$$$$`.

> **Claude Code** is Anthropic's coding assistant that runs in your terminal. A **skill** is a set of instructions you hand it on demand, by typing a `/` command. A **subagent** (or helper agent) is a separate copy of the assistant that the main session starts to do one job and report back. The main session is called the **orchestrator**.

## The skill

| Command | What it does |
|---|---|
| `/mock-use-subagents-wisely $$$ <task>` | Switches the session into lean-orchestrator mode at the given level, then starts the task. The mode stays on for every later message in the session. |
| `/mock-use-subagents-wisely` | Same, but asks you to pick a level first. If you do not answer within two minutes it picks `$$$`. |
| `$$` (or any run of `$`, on its own line) | Changes the level for the rest of the session. |

It is **manual only**. It never switches itself on.

## What it changes

Once on, the main session only plans, delegates, checks results and keeps track. Helper agents read the big files, write the code, run the tests and take the screenshots. This keeps the main session's memory small, so it stays sharp through a long piece of work.

Tiny requests are the exception. A one-line question or a two-line edit is done directly, because starting a helper for it would cost more than it saves.

The skill talks about three **model tiers**, never about model names, so it keeps working when models are renamed or replaced:

| Tier | Meaning | In Claude Code, late 2026 |
|---|---|---|
| **Top** | the most capable model, usually with its own tight usage limit | Fable |
| **Mid** | the everyday strong model | Opus |
| **Low** | the fast and cheap model | Sonnet |

At start the session looks at which models it can give a helper, ranks them by capability and cost, and tells you the mapping in one line. The same rule works in other coding agents, whatever models they offer.

## The levels

| Level | Top does | Mid does | Low does |
|---|---|---|---|
| `$` | nothing | all the work and the final review | easy work with a clear spec |
| `$$` | the final review, and a bug Mid failed to fix twice | everything else | easy work |
| `$$$` (default) | the final review, hard bugs, very complex packages, messy multi-file fixes | ordinary packages and bugs | easy work |
| `$$$$` | the above, plus anything touching core state, rendering or long-running jobs | easy and ordinary tasks | nothing |
| `$$$$$` | all non-trivial work | easy mechanical tasks only | nothing |

`$$$$$` is for the last hours before your usage limit resets, when unused Top budget would be lost anyway. Only you choose it.

At every level, only one Top helper runs at a time, parallel helpers never edit the same file, and one test suite runs at a time. When the work is done, a read-only reviewer reads the whole change for bugs and leftovers before you see it.

## What you need

- [Claude Code](https://claude.com/claude-code). Install it first; nothing here works without it.

Other coding agents work too, with their own helper-agent feature. See Option 4 below.

## Installing

Options 1 to 3 are for Claude Code. Option 4 is for other agents. Pick one.

### Option 1 — ask Claude Code to do it

This is the easiest way, and you do not need a terminal. Open Claude Code and paste this:

```
Please install the Claude Code skills from https://github.com/panuakdet-gmail/mock-use-subagents-wisely.

1. Download the repository, with its full history, to a temporary folder.
2. For each folder inside its skills/ folder, look for a folder with the same name in ~/.claude/skills/.
   - If there is none, copy the folder there.
   - If there is one, and every file in it matches a version that was once published in this repository, replace it with the new version.
   - If there is one, and it matches no published version, I have edited it. First move my copy to ~/.claude/skill-backups/<folder name>-<today's date>/. Then install the new version, show me what I had changed, and offer to merge my changes into it.
3. Delete the temporary download and tell me what you installed or updated.
```

Claude Code asks your permission before it changes anything. When it has finished, restart Claude Code.

**To update later**, paste the same prompt again. If you changed the skill yourself, your version is saved first, and Claude Code offers to merge your changes into the new one.

### Option 2 — install as a plugin

Choose this if you want Claude Code to handle updates for you. Inside Claude Code, run:

```
/plugin marketplace add panuakdet-gmail/mock-use-subagents-wisely
/plugin install use-subagents-wisely@use-subagents-wisely-skills
```

To update later, run `/plugin marketplace update use-subagents-wisely-skills`. Do not also use Option 1, otherwise the skill is installed twice.

### Option 3 — copy the folder yourself

In a terminal:

```bash
git clone https://github.com/panuakdet-gmail/mock-use-subagents-wisely.git
mkdir -p ~/.claude/skills
cp -R mock-use-subagents-wisely/skills/mock-use-subagents-wisely ~/.claude/skills/
```

Then restart Claude Code. To update, delete the downloaded folder and run the same commands again. This replaces the installed folder, so any changes you made to it are lost.

### Option 4 — ask another agent to do it

Open Codex, Antigravity, Gemini CLI, or whatever agent you use, and paste this:

```
Please install the skills from https://github.com/panuakdet-gmail/mock-use-subagents-wisely for this agent.

1. Download the repository, with its full history, to a temporary folder.
2. Each folder inside its skills/ folder is one skill. Install each folder whole, including subfolders such as assets/, because the instructions use those files.
3. The skills were written for Claude Code, so adapt only the packaging: register each one the way this agent handles reusable skills or commands, so I can run it by name. Do not change the instructions inside each SKILL.md.
4. Before you install each skill, check whether I already have it.
   - If I do not, install it.
   - If I do, and its files match a version that was once published in this repository, replace it with the new version. Ignore packaging changes that an agent made when it installed the skill.
   - If I do, and it matches no published version, I have edited it. First copy my version to a backup folder outside the place this agent loads skills from, and tell me where it is. Then install the new version, show me what I had changed, and offer to merge my changes into it.
5. Delete the temporary download. Then tell me what you installed or updated, and how to run each skill here.
```

**To update later**, paste the same prompt again.

## Using it

Start a piece of work with a level:

```
/mock-use-subagents-wisely $$ migrate the settings page to the new form library
```

The session answers with one line of mapping, then gets to work:

```
Level $$. Tiers: Top = fable, Mid = opus, Low = sonnet. Starting.
```

Later, raise the level when a hard bug shows up:

```
$$$$
```

Or leave the level out and let it ask:

```
/mock-use-subagents-wisely fix the flaky upload test
```

A dialog offers the five levels, with `$$$` recommended. If you walk away, it picks `$$$` after two minutes and starts.

## What it produces

Nothing on disk of its own. The output is the work you asked for, done by helpers, plus a short read-only review of the whole change at the end. Anything the reviewer finds is fixed before the session reports back.

## Licence

MIT, see [LICENSE](LICENSE). It covers the text in this repository. It says nothing about the idea of delegating work to helper agents, which is nobody's to license.
