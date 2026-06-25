# SETUP — the required setup and the operating rules

This is the one document that explains how the Source-of-Truth framework is wired, so the
process doesn't get brittle later. Read it once. The upfront steps are deliberate; everything
after them is the same two moves forever.

---

## The mental model (read this first)

Two tools that can't see each other do the work:

- **Claude chat = thinking.** It reads a repo and emits artifacts (a handoff or a changeset).
  It always pairs with a **SoT repo**.
- **Claude Code = building.** It writes code and keeps docs in sync. It always opens a
  **code repo**.

And two kinds of repo:

- **Templates** (`sot-framework`, `sot-framework-code`) — read-only *sources*. You never build
  inside them and never edit them. They exist to be copied.
- **Product repos** (`<slug>-sot`, `<slug>-code`) — a fresh copy per product, with their own git
  history. This is where your product actually lives.

> **The pointing rule (the thing to remember):**
> chat → a **SoT** repo · Claude Code → a **code** repo.
> For a brand-new product, the code repo you open is the **`sot-framework-code` template**.
> For every change after that, it's your product's **`<slug>-code`** repo.

---

## One-time setup

1. **Clone both templates as siblings** in one folder (any folder you like):

   ```
   gh repo clone CATrainer/sot-framework        # SoT template  (the brain)
   gh repo clone CATrainer/sot-framework-code   # code template (carries the sot-build skill)
   ```

   Keep them next to each other — the skill copies the SoT template by looking for it beside
   the code template:

   ```
   <your folder>/
     sot-framework/        ← SoT template
     sot-framework-code/   ← code template   (you open Claude Code here to start products)
   ```

2. **Authenticate the GitHub CLI** so Claude Code can create your product repos:

   ```
   gh auth login
   ```

That's it. You do not pre-create any product folders or repos — the skill does.

---

## Starting a NEW product

1. **Think it through in Claude chat.** Open a fresh chat, paste the contents of
   `sot-framework/framework/cold-start.md`, and describe your idea. Chat runs discovery, the
   walk, a design pass (in Claude Design), and emits a single **handoff file** — a `# PRODUCT:`
   header plus `# FILE:`-delimited sections — and tells you to download it.

2. **Hand it to Claude Code.** Open Claude Code **in your `sot-framework-code` folder**,
   **attach** the handoff file and your Claude Design `.zip`, and say:

   > *"Set up [Product] from these files and build the first task."*

3. **What the skill does** (no further input needed): reads the slug from the handoff, creates
   `<slug>/` next to your templates, copies both templates into `<slug>/<slug>-sot` and
   `<slug>/<slug>-code`, populates them from the attachments, gives each its own git history and
   a **private** GitHub repo via `gh`, and builds task 001.

You end up with:

```
<your folder>/
  sot-framework/            ← template (untouched)
  sot-framework-code/       ← template (untouched)
  <slug>/
    <slug>-sot/             ← your product's brain   (chat points here to iterate)
    <slug>-code/            ← your product's build    (Claude Code opens here to iterate)
```

---

## Changing an EXISTING product (iterate)

1. **In Claude chat:** fresh chat, paste `framework/iterate.md`, and **connect the chat to your
   `<slug>-sot` repo** (GitHub "Add from GitHub", read-only is fine) — the live SoT, *not* the
   template. Describe the change. Chat orients against current truth and emits a **changeset**.

2. **In Claude Code:** open your **`<slug>-code`** repo, **attach** the changeset, and say:

   > *"Apply this changeset and build the task."*

   The skill finds the `<slug>-sot` sibling, applies the edits in place, updates the index, and
   commits both repos.

---

## Keeping it un-brittle (the few things that matter)

- **Don't move the templates apart.** The skill finds the SoT template beside the code
  template; if it's missing it re-clones it from GitHub, but siblings is the happy path.
- **Don't edit the templates.** They're sources. If you want to improve the framework itself,
  change it in the template repos deliberately and push — that's a separate act from building a
  product.
- **Keep `gh` authenticated.** Repo creation depends on it; the skill stops and asks you to run
  `gh auth login` if it isn't.
- **Names come from chat, once.** Chat picks the product name and writes the slug into the
  handoff's `# PRODUCT:` header. Nothing downstream re-derives or guesses it, so it can't drift.
