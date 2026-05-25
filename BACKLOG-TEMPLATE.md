# [PROJECT_NAME] — Working backlog

_Copyright © {YEAR} {OWNER_NAME}. All rights reserved._

> [PROJECT_NAME] is built with Claude (Anthropic) as a continuous
> collaborator. The PRD, ARCHITECTURE doc, and most code are
> produced via human-AI pairing — the planning docs are written
> dense and self-contained so a fresh Claude session can cold-read
> and contribute immediately.

**Single source of truth** for everything deferred. Each dispatch
handoff's "Items deferred" sections feed this file; PRD points here
for the working list. Items move between horizons as priorities
shift.

---

## Horizons

- **Next** — actively being considered for the immediate next
  dispatch. Usually 1-3 items, often shaped by recent user
  feedback. Empty is a valid state.
- **Soon** — likely to come up in the near term (weeks). Not
  committed; can be moved out as priorities shift.
- **Later** — captured but no urgency. The big backlog. Includes
  items gated by specific triggers (e.g., "needs paid-tier
  infrastructure first"), items waiting on usage feedback to
  justify their effort, and genuine future-phase structural work.

(There are no version-number scheduling labels. The project deploys
continuously; version numbers don't earn their keep as horizons
when the BACKLOG already captures relative priority. Where a
milestone-style boundary matters — e.g., "before opening signup
to the public" — capture it in the item's note rather than as a
section heading.)

## Size tags

- **XS** — ~10-30 lines. Often skip the dispatch layer; nautilus
  edits directly.
- **S** — ~50-150 lines. Single dispatch, no pre-read needed in
  principle (but pre-read is cheap and catches real issues — see
  WORKING-MODEL.md §3).
- **M** — ~200-500 lines. Single dispatch, pre-read recommended.
  *First size where planning matters before dispatch.*
- **L** — ~500-1500 lines. Single dispatch, pre-read review
  required. *Intentional planning; often warrants a separate
  design doc first.*
- **XL** — >1500 lines OR new architectural pattern. *Split into
  multiple dispatches OR treat as a phase boundary.*

XS and S are roughly equivalent in dispatch cost (both fast, both
low-risk). M and L are where intentional planning earns its keep.
L and XL are where you start thinking about refactoring
opportunities or splitting work.

## Status conventions

- `[ ]` not started
- `[~]` design captured, implementation not started
- `[›]` in flight (dispatch active)
- `[x]` done (moves to Done section eventually)

---

## Next

(empty — populate as items earn promotion from Soon)

---

## Soon

Likely to come up in the next few weeks, in roughly this order
(smallest first, so quick wins land before bigger commitments).

(empty — populate as design conversations earn items their place)

---

## Later

The big backlog. Items gated by specific triggers, items waiting
for usage feedback to validate demand, and future-phase structural
work. Move to Soon as triggers fire or priorities shift.

(empty — populate as items get filed during dispatches, design
conversations, or post-ship discoveries)

---

## Done

For historical context — items that started in this backlog (or
in handoffs) and shipped.

(empty — populate as items ship; each Done entry gets a substantive
summary line so future readers can understand what landed and when)

---

## Maintenance

- **When a new dispatch handoff lists "Items deferred"** → also
  append them here (HANDOFF-TEMPLATE.md reminds the cuttlefish).
- **When a backlog item ships** → move to Done section with a
  short note.
- **When user feedback arrives** → re-rank the Next section;
  promote items from Soon as priorities shift.
- **Periodic cleanup** (~quarterly): review Later section for
  items that have genuinely aged out and can be dropped.
- **If something obviously XL ends up in Later**, name the
  splitting plan inline ("likely 3 dispatches: A, B, C") so a
  future scope-picking conversation has the breakdown ready.
