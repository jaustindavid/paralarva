# [PROJECT_NAME] — AGENTS.md

_Copyright © {YEAR} {OWNER_NAME}. All rights reserved._

> [PROJECT_NAME] is built with Claude (Anthropic) as a continuous
> collaborator. The PRD, ARCHITECTURE doc, and most code are
> produced via human-AI pairing — the planning docs are written
> dense and self-contained so a fresh Claude session can cold-read
> and contribute immediately.

This file is the entry point for any coding agent (Claude Code,
Cursor, Aider, Codex, etc.) picking up work on [PROJECT_NAME].
Read it first. It is intentionally short; the canonical docs do
the heavy lifting.

---

## Required reading, in order

1. **[PRD.md](PRD.md)** — what to build. Data model, access
   control, user flows, cost spec, milestones.
2. **[ARCHITECTURE.md](ARCHITECTURE.md)** — how to build it.
   Module layout, key abstractions, deploy/env, testing.

Do not start writing code until both are read. The planning docs
are written for cold-read; budget ~30 minutes.

---

## The load-bearing sections

(For new projects: fill in once the PRD + ARCHITECTURE settle.
Typically the most-often-broken pieces or the load-bearing
contracts go here. Examples from prior projects:)

- **[PRD §X — Cost control specification](PRD.md#X)** —
  per-active-user cost must stay under $Y/mo. The Z debounce and
  Q-only recompute are non-negotiable.
- **[ARCHITECTURE §X — Core state machine](ARCHITECTURE.md#X)** —
  the most-often-broken piece. Race-safe via stable IDs;
  mandatory unit tests live here.

Violating either is violating v1's primary commitments.

(Delete this section's parenthetical guidance once the load-bearing
items for your project are identified and filled in.)

---

## Hard guardrails — do not cross without asking

Each is a deliberate "no" in v1. Each has a rationale — link to it
in ARCHITECTURE.md or PRD.md.

(Defaults that have proven across prior projects — keep or remove
as appropriate for your project's posture:)

- **No Cloud Functions or server code.** Anything not enforceable
  in client-evaluated security rules is out of scope for v1.
  (Override only when paid features or webhook integrations need
  it — both genuine signals.)
- **No real-time database listeners** (`onSnapshot` in Firestore,
  WebSocket subscriptions, SSE, etc.). Use one-shot reads. The
  collaboration model is async-by-explicit-action, not
  push-based. (Override only when a feature genuinely needs
  live-updating UI.)
- **No external state library** (Redux, Zustand, etc.). React
  Context plus local state is sufficient. (Override only when
  state coordination becomes genuinely cross-cutting and
  Context patterns get unwieldy.)
- **No SSR.** Static SPA. (Override only when SEO becomes a real
  product need.)
- **No `any` in TypeScript** without an inline comment explaining
  why.
- **No checked-in secrets.** API keys go in `.env.local`,
  gitignored.
- **No behavioral tracking / analytics SDKs** unless the PRD's
  privacy posture explicitly permits them. (Default posture for
  privacy-respecting projects.)

Project-specific guardrails go here as they're established.

---

## Commit & PR hygiene

- One PR per dispatch (or smaller). Each PR runnable and
  type-clean.
- Dispatches are defined as briefs in `dispatch/`; the matching
  handoff lands alongside.
- Human review on every PR. No agent self-merge — partly for code
  quality, partly for IP clarity on AI-collaborative work.
- Strict TypeScript throughout (if TS project). Conventional
  commit messages preferred but not enforced.

---

## Testing expectations

(Fill in per project. Common minimums for prior projects:)

- Unit tests for the core reducer / state machine
- Unit tests for any pure-function utilities (formatters,
  parsers, serializers)
- Rules tests if using Firestore + security rules (emulator-based)

Deferred to component-test-infrastructure dispatch (if applicable):
component / integration tests. Use the manual checklist in each
dispatch's acceptance criteria instead.

---

## Linting

Markdown is linted with `markdownlint-cli2`. Config lives at
`.markdownlint.jsonc` at the repo root.

Run before considering any markdown change complete:

```sh
npx markdownlint-cli2 "**/*.md"
```

Must exit clean (zero errors). The 80-column rule is enforced via
MD013; tables, code blocks, and ASCII diagrams are exempt by
design. If you intend to suppress a rule for a specific reason,
add it to `.markdownlint.jsonc` with a comment explaining why —
do not use inline disable comments.

Code linting (ESLint or equivalent) per project setup.

---

## If unsure, ask

Pause and ask the human before:

- Adding a new top-level dependency.
- Introducing any backend (Cloud Function, server, edge worker,
  etc.) that isn't already specified in ARCHITECTURE.md.
- Changing the database schema.
- Touching the security rules.
- Picking a third-party service or SDK not already in
  ARCHITECTURE.md.
- Resolving any of the open questions in PRD §11.2.

The cost of pausing is low; the cost of a reverted PR is higher.

---

## Project identity

[PROJECT_NAME] is a **private, commercial project** (or: a
**personal project**, or: **open-source under [LICENSE]** — adjust
to reality). All rights reserved (or: per LICENSE).

(Privacy-posture statement, if applicable: "Treat planning docs
and code accordingly: no public posting of internals, no copying
to other repos, no inclusion in training-data exports. If unclear,
ask.")

---

## When you create a new artifact

Every top-level markdown deliverable in this project (PRD,
ARCHITECTURE, README, AGENTS, design docs, RFCs, etc.) must open
with two lines under the title, in this order:

1. The copyright header:
   `_Copyright © {YEAR} {OWNER_NAME}. All rights reserved._`
2. The AI-first preamble blockquote (the same one that opens this
   file, with `[PROJECT_NAME]` substituted in).

Match the surrounding doc's style for everything else. Apply
automatically — the human will not remember to ask.

---

## Working model

The project follows the **cuttlefish/nautilus** working model. See
the `paralarva/` source folder (or wherever the kit docs were
copied from) for:

- `CUTTLEFISH-NAUTILUS.md` — the conceptual frame
- `WORKING-MODEL.md` — the operational playbook (lifecycle,
  pre-read, antipatterns, stall recovery, post-ship fix protocol)
- `HANDOFF-TEMPLATE.md` — handoff doc structure

These docs are referenced from the project's dispatches and
should be considered required reading before contributing to any
dispatch's brief or handoff.
