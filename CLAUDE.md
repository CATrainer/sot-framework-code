# CLAUDE.md — Code Repo

This is the **code repo** for this product. Its paired **Source of Truth (SoT)** repo holds
the specs, decisions, and tasks that drive what gets built here.

## The pairing (how the two repos find each other)

By convention the two repos are **siblings in one product folder**:

```
[product]/
  [product]-sot/     ← the SoT repo (specs, decisions, tasks)
  [product]-code/    ← this repo (the skill lives in .claude/skills/)
```

The SoT is at `../[product]-sot/`. If you keep the repos elsewhere, set the path here:

> **SoT path:** `../sprout-sot`  *(edit if not using the sibling convention)*

## The bus protocol

The SoT repo is the **shared memory** between Claude chat and Claude Code. Chat reads the
SoT and emits artifacts; this side (Claude Code) executes and writes back. Neither tool
remembers the other's work — the SoT is the only shared state.

**Before building anything:**
1. Read `../[product]-sot/INDEX.md` and `../[product]-sot/foundation.md`.
2. Read only the area file(s) the current task names — not the whole SoT.
3. Read `../[product]-sot/design/direction.md` for locked visual tokens.

**After building:**
1. Write any implementation decisions back into the relevant SoT area file's `## Decisions`.
2. Update `INDEX.md` status in the same operation.
3. Move the completed task to `tasks/done/`.

All of this is enforced by the `sot-build` skill in `.claude/skills/` — invoke it for any
SoT-driven build or changeset.

## design-reference/

On first build, the user unzips their Claude Design project here as scaffolding. Extract
what you need to match the design, governed by `../[product]-sot/design/direction.md` (the
durable truth). After the first build, `design-reference/` is disposable — the built UI and
the tokens in the SoT are the source of truth, not this folder.

## Stack

Defined in `../[product]-sot/foundation.md`. Do not assume a stack — read it.
