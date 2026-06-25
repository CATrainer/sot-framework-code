---
name: sot-build
description: >
  Stand up a new framework product from a chat handoff, and execute SoT-driven builds and
  changesets thereafter. Use when handed a dumped handoff file (+ design zip) to set up a new
  product, when asked to build/apply a task, or to set up the SoT from artifacts. Enforces the
  repo-as-bus protocol: copies the templates into fresh per-product repos, reads narrow, writes
  back to both repos, applies the anti-bloat self-healing law, and keeps INDEX.md current.
  MANDATORY TRIGGERS: set up <product> from these files, set up the SoT, build the task, apply
  this changeset, build from the handoff, run the spec.
---

# sot-build

Enforce the protocol that keeps a product's two repos (SoT + code) in sync — and stand up
those two repos in the first place from the chat handoff. The SoT repo is the shared memory
between Claude chat (which thinks) and Claude Code (which builds); this skill is the
execution layer.

## Pick the mode first

- **Cold setup** — you're running **inside the `sot-framework-code` template** and the user
  has **attached** a handoff file (+ design `.zip`) for a product whose repos **do not exist
  yet**. Create them from the templates. Go to **Setup**. (Running from the code template is
  what makes this skill available before any product repo exists.)
- **Existing project** — you're in a real product's `<slug>-code` repo with a paired
  `<slug>-sot` sibling; you're building a task or applying an attached changeset. Go to
  **Guard → Orient**.

If unsure which: cwd is a template (`sot-framework-code`) and inputs are attached → cold setup;
a paired product SoT repo with an `INDEX.md` already exists → existing project.

---

## Setup — create a new product's repos from the templates

Run this the first time a product is handed off. **You are running inside the code template
repo (`sot-framework-code`)** — that's how this skill is available before any product repo
exists. The user has **attached** the handoff and design to the message. You do the rest.

1. **Read the attached inputs — they are attachments, not files to hunt for.**
   - The **handoff** is a single markdown file: a `# PRODUCT: <Name> · slug: <slug>` header,
     then sections delimited by `# FILE: <relative/path>` lines (`foundation.md`, one file per
     product/business area, `design/direction.md`, `tasks/open/001-*.md`).
   - The **design** is a Claude Design `.zip` (or, accepted as a fallback, a path to one).
   - If the handoff isn't attached, or has no `# PRODUCT:` header, **ask once** for it. If the
     design zip is missing, **ask once**. **Never guess and scaffold from the guess** — that
     produces duplicate folders and wasted searching.
2. **Take the slug from the handoff header** — use it verbatim as `<slug>`. Do not invent or
   re-derive a name; chat already chose it.
3. **Create the product folder and copy both templates into it.**
   - Product folder: `<slug>/`, created as a **sibling of the templates** (i.e. next to
     `sot-framework-code/` and `sot-framework/`), unless the user said otherwise.
   - `<slug>/<slug>-code` ← copy of **this** repo (the code template, your cwd).
   - `<slug>/<slug>-sot`  ← copy of the **SoT template**: the sibling `../sot-framework` if
     present, else `gh repo clone CATrainer/sot-framework`, else **ask once** for its path.
   - **Delete the copied `.git`** from each — the product gets its own history, not the
     framework's. Exclude `design-reference/` contents from the code copy if any leaked in.
4. **Populate the SoT from the handoff** (see *Populate* below) and **unzip the design** into
   `<slug>/<slug>-code/design-reference/`.
5. **Give each repo its own git + private remote.** In each: `git init`, stage, baseline
   commit, then `gh repo create <slug>-sot --private --source=. --remote=origin --push`
   (and `<slug>-code`). **Private by default** — this is a real product, not a template. If
   `gh` is not authenticated, **stop** and tell the user to run `gh auth login`.
6. From here on, work **in `<slug>/<slug>-code`** (the product, not the template). Proceed to
   build task 001 (**Execute → Write back → Report**).

### Populate — split the handoff into the SoT files
- Split the handoff on its `# FILE: <path>` delimiters and write each section to that path,
  **replacing the Sprout example entirely** — delete the template's `foundation.md`,
  `product/reminders.md`, `business/pricing.md`, `design/direction.md`, and
  `tasks/open/001-reminders-engine.md`, and write the handoff's versions in their place.
- A handoff "commitments" / business section lands in `business/` (e.g.
  `business/commitments.md`) — that folder is the home for business area files.
- **Rebuild `INDEX.md`** to the real areas and statuses (the index is sacred).
- The handoff file is now consumed. It's preserved in the baseline commit, so delete it from
  the working tree — working files hold only what is currently true.

---

## Guard — existing project only

Before building in an existing project, verify it's a framework project so the skill never
misfires on an unrelated repo:

1. Read this code repo's `CLAUDE.md` for the **SoT path** (default `../[product]-sot`).
2. Confirm `[sot-path]/INDEX.md` exists.

If `INDEX.md` is not found, **stop** and tell the user: "This doesn't look like a framework
project — I can't find the paired SoT repo's INDEX.md. Check the sibling-folder convention or
the SoT path in CLAUDE.md." Do not apply the protocol to a non-framework repo.

## Orient (read narrow)

Read, in order, and **only** what's needed:
- `[sot-path]/INDEX.md` — the map.
- `[sot-path]/foundation.md` — the bounded core.
- The area file(s) named by the task/changeset — not the whole `product/` or `business/`.
- `[sot-path]/design/direction.md` if the work touches UI.

Never load the entire SoT. Loading everything is what makes a model bury and contradict
known decisions.

## Execute

### If building a task
- Build against the area file's acceptance criteria. Honour every invariant in
  `foundation.md` and the area file.
- Match `design/direction.md` tokens. On first build, use `design-reference/` as the
  concrete render to match, then treat it as disposable.

### If applying a changeset (iteration)
- Apply edits **in place**. Where the changeset supersedes a decision, edit the old decision
  in place — do not append a contradicting entry. (See anti-bloat law below.)

## Write back (both repos, self-healing)

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
5. **Commit** both repos with a clear message, and push. Git is the history that makes
   pruning safe — so a product repo must always be a real git repo with a remote (Setup
   guarantees this).

## Report

Tell the user, briefly: what was built/changed, which SoT files were updated, the new index
status, and what the next open task is (if any). Don't recount the whole process — just the
result and the next step.

## Rules

- Copy the templates, then give the product **its own** git history and **private** remotes;
  never reuse the framework's `.git`.
- Locate handed-off inputs deterministically; ask once if ambiguous. Never guess-and-scaffold.
- Read narrow, never wide.
- Edit in place; never append a contradiction. Git holds history.
- The index is sacred — update it in the same operation, every time.
- Don't assume a stack or a design — read them from the SoT.
- If a build reveals a contradiction with a locked decision, stop and surface it as a
  backward edge; don't silently mutate foundation.
