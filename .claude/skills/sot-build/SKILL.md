---
name: sot-build
description: >
  Execute SoT-driven builds and changesets for a framework product. Use when handed a
  handoff file or changeset from Claude chat, when asked to set up a SoT repo from
  artifacts, or when asked to build/apply a task. Enforces the repo-as-bus protocol: reads
  narrow, writes back to both repos, applies the anti-bloat self-healing law, and keeps
  INDEX.md current. MANDATORY TRIGGERS: set up the SoT, build the task, apply this
  changeset, build from the handoff, run the spec.
---

# sot-build

Enforce the protocol that keeps a product's two repos (SoT + code) in sync. This skill is
the execution layer; the SoT repo is the shared memory between Claude chat (which thinks)
and Claude Code (which builds).

## Step 0 — Guard: is this a framework project?

Before doing anything, verify this is a framework project so the skill never misfires on an
unrelated repo:

1. Find the SoT path: read this code repo's `CLAUDE.md` for the **SoT path** (default
   `../[product]-sot`).
2. Confirm `[sot-path]/INDEX.md` exists.

If `INDEX.md` is not found, **stop** and tell the user: "This doesn't look like a framework
project — I can't find the paired SoT repo's INDEX.md. Check the sibling-folder convention
or the SoT path in CLAUDE.md." Do not apply the protocol to a non-framework repo.

## Step 1 — Orient (read narrow)

Read, in order, and **only** what's needed:
- `[sot-path]/INDEX.md` — the map.
- `[sot-path]/foundation.md` — the bounded core.
- The area file(s) named by the task/changeset — not the whole `product/` or `business/`.
- `[sot-path]/design/direction.md` if the work touches UI.

Never load the entire SoT. Loading everything is what makes a model bury and contradict
known decisions.

## Step 2 — Execute

### If setting up a SoT repo from a fresh handoff file
- The template ships with the neutral example (Sprout). On first real setup, **replace the
  example content entirely** with the handoff's content — don't merge with Sprout.
- Write `foundation.md`, each product/business area file, `design/direction.md`, and the
  first task into the SoT repo in their correct formats (see the SoT repo's `CLAUDE.md`).
- Rebuild `INDEX.md` to reflect the real areas and statuses.

### If building a task
- Build against the area file's acceptance criteria. Honour every invariant in
  `foundation.md` and the area file.
- Match `design/direction.md` tokens. On first build, use `design-reference/` as the
  concrete render to match, then treat it as disposable.

### If applying a changeset (iteration)
- Apply edits **in place**. Where the changeset supersedes a decision, edit the old decision
  in place — do not append a contradicting entry. (See anti-bloat law below.)

## Step 3 — Write back (both repos, self-healing)

After building or applying:

1. **Decisions** → write implementation decisions into the relevant area file's
   `## Decisions` block. The *why* lives next to the thing it governs.
2. **Anti-bloat law** → every write is also a prune. New truth **replaces** old truth in
   place. Superseded reasoning is preserved by git history, not by leaving stale entries.
   Never keep a contradicting pair. Keep `foundation.md` under ~2 pages — push area-specific
   content down if it grows.
3. **INDEX.md** → update area status (e.g. Spec'd → Built) in the **same operation**. A
   stale index is a failed operation. If you created/renamed/retired an area, the index edit
   is mandatory now.
4. **Task** → move the completed task from `tasks/open/` to `tasks/done/`.
5. **Commit** both repos with a clear message. Git is the history that makes pruning safe.

## Step 4 — Report

Tell the user, briefly: what was built/changed, which SoT files were updated, the new index
status, and what the next open task is (if any). Don't recount the whole process — just the
result and the next step.

## Rules

- Read narrow, never wide.
- Edit in place; never append a contradiction. Git holds history.
- The index is sacred — update it in the same operation, every time.
- Don't assume a stack or a design — read them from the SoT.
- If a build reveals a contradiction with a locked decision, stop and surface it as a
  backward edge; don't silently mutate foundation.
