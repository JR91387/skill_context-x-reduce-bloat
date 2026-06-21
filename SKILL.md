---
name: context-x
description: Context-X is like Gas-X for your context window. Offloads reference information to a loadout in .claude, compresses and refines remaining prose, prunes MEMORY.md and trims verbose skill descriptions. Also: /gasx, /contextx, /tokenbloat, /reducebloat, /reducecontext.
user-invocable: true
---

# /context-x — Context Bloat Relief

Arguments: `$ARGUMENTS`

`--restore` reverts last run from `backup_originals/`. No other args needed — level drives scope.

## Pre-flight

Run `/context`. Identify heaviest modifiable buckets. Skip unconditionally: System Prompt, System Tools, MCP tools at 0 tokens — unmodifiable.

Select reduction level with AskUserQuestion (single-select):
- **Minimal** — CLAUDE.md move-to-loadout only; scope = claude only
- **Normal** — CLAUDE.md move + compress prose; memory prune; scope = claude + memory
- **Deep clean** — full compression; skill descriptions trimmed; scope = all targets
- **Maximum** — max aggression; single-line bullets; scope = all targets; requires confirm before write

Level drives scope automatically. No separate scope selector.

---

## Setup — scaffold loadout if missing

Before any phase, check for `.claude/loadout/`. If absent, create:

```
.claude/loadout/
  architecture/       build, deploy, db, backend, frontend specs
  decisions/          locked decisions + rationale
  sessions/           session artifacts, plans
  memories/           reference facts moved out of CLAUDE.md
  operations/         scripts, maintenance, rare-op guides
  backup_originals/   pre-edit backups
  index.yaml          on_demand manifest stub
```

`index.yaml` stub:
```yaml
_help: "Register on-demand loadout files here. Claude reads them when task needs them, not at session start."
load:
  always: []
  on_demand: []
```

Global scope (`~/.claude/loadout/`): same structure, no `index.yaml`.

---

## Phase 1 — CLAUDE.md reduction + compression

**Atomicity rule:** write ALL loadout files first. Only then rewrite CLAUDE.md in one pass. Never leave CLAUDE.md with pointers to unwritten files.

Back up first: copy `CLAUDE.md` → `backup_originals/CLAUDE.md.original.yyyymmdd`. Never compress `*.original.*`.

For each CLAUDE.md in scope — two passes:

### Pass A: Move reference content to loadout

**KEEP** (loads every session):
- Absolute behavioral rules: machine labels, safety constraints, no-X-on-Y rules
- Compact identity/access/machine tables (≤5 rows)
- Critical naming conventions affecting every edit
- Machine names, IPs, ports, service references
- Anything causing wrong action if forgotten mid-task

**MOVE** to loadout:
- How-to instructions: build, deploy, db connect, git
- Background, rationale, history
- Long reference tables: schemas, crates, models, domains
- Rare-operation guides
- Lookups, not behavior rules

Loadout subfolder by content type:
- `architecture/` — build, deploy, db, backend, frontend specs
- `decisions/` — locked decisions, rationale, history
- `operations/` — scripts, maintenance, rare-op guides
- `memories/` — reference facts, lookup tables

Replace each MOVE block with: `<Topic> detail: see <absolute path>`

Register each new file under `on_demand:` in `index.yaml` (project scope only).

### Pass B: Compress remaining prose

Apply to all prose remaining in CLAUDE.md after Pass A.

**Drop:** filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course), hedging (it might be worth/you could consider), redundant phrasing (in order to → to, make sure to → ensure), connective fluff (however/furthermore/additionally).

**Articles (a/an/the):** drop only when clearly decorative. Never drop before proper nouns, technical terms, or singular definite referents where the/a distinction carries meaning.

**Short synonyms:** where meaning is identical — big not extensive, fix not implement a solution for, use not utilize. Fragments OK. State actions directly.

Merge redundant bullets only when they say the *exact* same thing differently. Never merge bullets with distinct commands or steps.

**Never touch — preserve exactly:**
- Code blocks, inline code, file paths, commands, URLs
- Technical identifiers: paths, port numbers, env vars (`$HOME`, `db_port=5432`), command names (`/dev/tty`), protocol names — never shorten even if they look like fragments
- Technical terms, library/API/protocol names, proper nouns
- Dates, version numbers, YAML/frontmatter
- Safety rules and absolute constraints — compress surrounding prose, not the rule itself
- Markdown tables and nested list structure

Scope: `.md` files only. Mixed files: compress prose, leave code blocks unchanged.

**Maximum level only:** before writing, show a change summary (sections moved, bytes saved estimate) and require explicit user confirm via AskUserQuestion.

---

## Phase 2 — MEMORY.md pruning

MEMORY.md index always loads. Body files load only when explicitly Read.

Back up all body files that will be modified: copy to `backup_originals/memory/` before any overwrite.

### Prune

Remove entry ONLY when ALL hold:
- Resolved, no open follow-up
- No ongoing behavioral constraint
- Fact captured in CLAUDE.md or loadout

Never prune if:
- Changes behavior on any future turn
- Names specific command, path, role, or field hard to re-derive
- Resolution status uncertain

`rm` may be hook-blocked — use overwrite-to-stub:
1. Remove index line from MEMORY.md
2. Overwrite body: `REMOVED — [reason] — de-indexed [date]`

**Stubs survive at all levels including Maximum.** Never delete a `REMOVED — ...` entry.

### Compress retained entries

- Drop: narrative history, incident prose, "we got burned when" examples
- Keep: rule, trigger condition, specific constraint names/paths/values
- **Specificity is load-bearing**: field names, CLI commands, role names, URLs survive
- Technical identifiers never shortened (same rule as Pass B)
- Test: "would removing this cause future session to mis-apply rule?" — if yes, keep it

---

## Phase 3 — Skill description trimming

*(Deep clean and Maximum levels only)*

Descriptions load every session for auto-invocation — savings compound across all future sessions.

For each `~/.claude/skills/*/SKILL.md`:
1. Read `description:` field
2. Keep: trigger keywords, scope limits, tool names, alias list
3. Cut: elaboration of what trigger words already imply, examples, filler
4. `Also invocable as /foo` → `Also: /foo`
5. Technical identifiers never shortened
6. Test: "would removing this cause mis-invocation?" — if yes, keep it
7. AskUserQuestion per skill before applying; one skill at a time

---

## --restore

If `--restore` passed as argument:
1. List contents of `backup_originals/` with timestamps
2. AskUserQuestion: which backup to restore (or cancel)
3. Copy selected backup back to original location
4. Report what was restored

---

## Verification (after each phase)

1. Re-read modified CLAUDE.md — every safety rule present and unambiguous
2. Each pointer: Read referenced loadout file, confirm exists and is readable (not just present)
3. Each `index.yaml` on_demand entry: Read and confirm accessible
4. Each pruned memory entry: body file is stubbed, not just de-indexed
5. Report results (see below)

---

## Results

After all phases complete, output:

**Before / after table** — one row per file touched:

| File | Before (chars) | After (chars) | Δ | Δ% |
|---|---|---|---|---|
| `~/.claude/CLAUDE.md` | — | — | — | — |
| `collection/CLAUDE.md` | — | — | — | — |
| `MEMORY.md` index | — entries | — entries | — | — |
| `skills/<name>/SKILL.md` | — | — | — | — |

Use char counts for prose files (CLAUDE.md, skill descriptions). Use entry counts for MEMORY.md index rows. Omit rows for files not touched.

**TL;DR summary** — 2–3 sentences max:

What was moved, what was compressed, what was pruned. Net session-start token impact. Any files skipped or decisions deferred to the user.

Example:
> Moved 6 sections from collection/CLAUDE.md to loadout (architecture, deploy, db, backend, insights, conventions). Compressed remaining prose 38%. Pruned 3 resolved memory entries. Session-start CLAUDE.md load drops ~71% (14k → 4k chars). Skill descriptions unchanged — no Deep clean or Maximum level selected.
