# Changelog — paralarva kit

> This document is part of the **paralarva** bootstrap kit — the
> working patterns used to spin up projects with Claude as
> collaborator. Planning docs in projects derived from this kit are
> written dense and self-contained so a fresh Claude session can
> cold-read and contribute immediately.

Tracks meaningful changes to the kit's planning docs (WORKING-MODEL,
templates, the cuttlefish/nautilus framing). Entries are newest-first.
Each records **why** the change was made and, when relevant, what was
**deliberately not propagated** from a project-specific source — so a
future maintainer doesn't re-add something that was intentionally left
out. Patterns usually originate in a live project, prove out across a
dispatch arc, then get promoted here.

---

## WORKING-MODEL.md — 2026-05-29: cuttlefish as full-lifecycle participants

**Why**: the doc cast cuttlefish as downstream-only (implementer +
pre-read reviewer), with design living purely in owner↔nautilus
chat. In practice cuttlefish now work upstream too — read-only
fan-out research/exploration and design-option generation that
feed the design conversation. The old framing under-sold them in
exactly the way the cuttlefish/nautilus paper warns against.
Originated in flog's rework; validated across route7's dispatch
arc; promoted to the kit.

**Conceptual changes** to the framing:

- **§1 Cast**: cuttlefish reframed as the *dispatched-agent
  class*, spawned per role (research / design / reviewer /
  implementer) — same agent class, different prompt + bounded
  task. Dropped the "two cuttlefish roles" enumeration;
  generalized "never communicate" across all roles.
- **§2 Lifecycle**: starts earlier —
  `research/exploration → fold → design conversation (optionally
  design-cuttlefish-fed)` — and names **owner review (V2)** as an
  explicit terminal step.
- **§3** renamed *"Dispatching cuttlefish (research, design,
  pre-read)"*: consolidates the former Pre-read + Fact-finder +
  Research-cuttlefish sections under one shared discipline
  (purpose-built prompt, bounded task, read-only, conclusions-not-
  dumps, nautilus folds). Fact-finder is now a named *flavor* of
  research; the verdict-running pattern (SHIP / DEFER / DROP) is
  the heaviest form of design support.

**New canonical conventions** (§5):

- **Model tier fits the risk** — cheap/fast tier for mechanical
  work, stronger tier for security / rules / data-model / money-
  touching surfaces and their pre-reads; stated explicitly in the
  brief's model line, default-up when unsure.
- **Owner review (V2)** — named as a first-class terminal
  lifecycle step, not an afterthought; V2 findings are first-class
  (XS folds inline, larger become dispatches).

**Renumbering**: three sections collapsed into §3, so §6→§4 …
§13→§11; all internal cross-refs remapped. Side effect: the
antipattern subsection numbering (6.1–6.10) is now correct
(antipatterns is genuinely §6).

**Deliberately NOT propagated from flog** (project-specific):

- The literal gate command list — §5 now says "lint, markdownlint,
  unit/rules tests, build; exact commands are project-specific
  (see AGENTS.md / package scripts)."
- File-path layout — the kit keeps its `dispatch/` convention;
  flog moved BACKLOG / WORKING-MODEL / HANDOFF-TEMPLATE to repo
  root, which is a flog choice, not a kit default.

**Known wart**: §3 subsections are in reference order (Design,
Pre-read, Fact-finder, Research-with-verdict), not strict
lifecycle order; the intro flags this. A clean lifecycle reorder
(Research → Design → Pre-read) is a deliberate future pass.
