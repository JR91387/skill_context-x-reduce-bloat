# Context-X — Gas-X for Your Context Window

![Context-X skill overview](ContextX_GasX_Content_Bloat_Skill.png)

A [Claude Code](https://claude.ai/code) skill that reduces session-start token bloat. When your `CLAUDE.md` and memory files are eating your context budget, `/context-x` fixes it.

## What it does

- **Offloads** reference-only sections from `CLAUDE.md` to on-demand `.claude/loadout/` files — only loaded when a task actually needs them
- **Compresses** remaining prose: drops filler, hedging, redundant phrasing; never touches code, paths, safety rules, or technical identifiers
- **Prunes** resolved entries from `MEMORY.md`
- **Trims** verbose skill `description:` fields that load every session

Typical result: 60–80% reduction in `CLAUDE.md` session-start token cost.

## Install

Copy `SKILL.md` into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/context-x
cp SKILL.md ~/.claude/skills/context-x/SKILL.md
```

Start a new Claude Code session to pick up the skill.

## Usage

```
/context-x
```

Also invocable as: `/gasx` · `/contextx` · `/tokenbloat` · `/reducebloat` · `/reducecontext`

The skill opens with a level selector:

| Level | What it does |
|---|---|
| **Minimal** | Move reference content to loadout only |
| **Normal** | Move + compress prose + prune memory |
| **Deep clean** | Full compression + skill description trimming |
| **Maximum** | Max aggression on all targets; confirms before writing |

Run `/context` first to see where your tokens are going.

## Restore

```
/context-x --restore
```

Reverts the last run from `backup_originals/`. All modified files are backed up before any changes are made.

## Requirements

- [Claude Code](https://claude.ai/code) (any recent version)
- Skills must be enabled in your Claude Code settings
