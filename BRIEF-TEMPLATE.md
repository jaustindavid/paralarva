# Dispatch brief template

> This document is part of the **paralarva** bootstrap kit — the
> working patterns used to spin up projects with Claude as
> collaborator. Planning docs in projects derived from this kit are
> written dense and self-contained so a fresh Claude session can
> cold-read and contribute immediately.

This template captures the shape of a good dispatch brief — the
artifact the nautilus produces and the implementer cuttlefish
executes. The structure is conventional, not rigid; deviate when
the dispatch genuinely needs different shape, but start from this
shape and have a reason for changes.

For the handoff doc (what the cuttlefish produces at the end),
see `HANDOFF-TEMPLATE.md` — the brief and handoff form a matched
pair, request and response.

---

## 1. Two load-bearing brief-authoring principles

### 1a. Describe requirements, not click paths

For any dispatch step that touches an external UI (GCP Console,
Firebase Console, AWS Console, Stripe Dashboard, GitHub Settings,
etc.):

- **Lead with the requirement** — what state the system must be in
  after this step. Stable over time.
- **Optionally include a dated breadcrumb** — the current UI path
  for finding the relevant controls, marked with the date last
  verified. Stale over time.
- **The executor finds requirement-shaped controls** through the
  current UI's labels and tabs. The brief sets the destination;
  the executor navigates.

Concrete shape:

```text
G3 — OAuth consent screen

Requirement: External user type, Testing mode, no logo uploaded,
test users = family Gmails, support email is a Google-managed
mailbox.

Current path (YYYY-MM-DD): APIs & Services → OAuth Consent
Screen → Overview → Get Started; test users under Audience tab.
```

**Why**: external UIs reorganize constantly. Click-path briefs go
stale fast; requirement-shaped briefs stay correct forever. A
single execution session can encounter 5+ UI drifts; the
requirements survive them all. (Empirical: flog's M1 hit 7+ UI
drifts during one afternoon.)

This principle applies to ALL external services. If you find
yourself writing "click Add → choose dropdown → select option,"
stop and rewrite as "the system must be in state X after this
step."

### 1b. Cold-readable

The brief is for an implementer cuttlefish with **no conversation
history**. Every reference must be reconstructable from the brief
itself (or the cited supporting docs). No "as we discussed,"
no implicit context, no shorthand that requires knowing what
the nautilus was thinking.

Test mentally: "If a fresh Claude session reads only this brief +
its required-reading list, can they execute?" If no, add the
missing context to the brief.

---

## 2. Canonical brief structure

Most dispatches follow this shape. Sub-headings are fine to add;
the major section order should stay consistent so future
nautilus / cuttlefish reading 10 briefs can navigate them
similarly.

### `# Project — <Dispatch name>`

Standard preamble (copyright + AI-first blockquote — see
HANDOFF-TEMPLATE §3).

One-paragraph synopsis: what this dispatch ships, what size it
is (XS/S/M/L/XL — see BACKLOG-TEMPLATE size tags), why now.

### `## 1. What you're starting with`

Cold-reader context: the state of the codebase as of this
dispatch's start. Reference relevant prior handoffs. Surface
anything load-bearing that the cuttlefish needs to know but
might not gather from the file system alone.

### `## 2. Required reading`

Numbered list of files the cuttlefish reads before writing any
code. Order matters — put highest-context-yield first. Include:

- AGENTS.md (always)
- Relevant prior handoffs (for context)
- The BACKLOG entry for this dispatch (often most of the design)
- Source files the dispatch will modify (with line numbers if
  specific functions matter)
- Test files the dispatch will extend

Cite specific line numbers where they help orient
(`src/foo.ts:42-58 — the function you'll extend`).

### `## 3. Scope`

The substantive section. Break into lettered subsections
(`### A. ...`, `### B. ...`) — usually one subsection per
"thing to do." Each subsection includes:

- What changes
- Why (briefly, if non-obvious)
- Code snippets where shape matters
- Cross-references to other subsections where coupling exists

End with `### Z. Out of scope` — explicit list of things this
dispatch does NOT do, with brief rationale for each. Prevents
scope creep during implementation.

### `## 4. Hard guardrails`

- Files in play (the rough count + listing — implementer should
  flag deviations)
- Files NOT to touch (the explicit exclusions; usually overlaps
  with PRD / ARCHITECTURE / AGENTS / lint configs)
- No new top-level dependencies (or explicit list of additions
  needed)
- 80-col line width
- Pass-all-existing-tests requirement

### `## 5. Acceptance criteria`

Numbered or prefix-grouped (e.g., S1-3, R1-5, U1-4, G1-5) list
of testable assertions. Each should be checkable by reading the
code + running gates. The handoff doc's Status section refers
back to these by their prefixes.

Always include a "Gates" subsection at the end with the standard
commands (`npm test`, `npm run lint`, `npm run build`, etc.) —
all must pass before handoff.

### `## 6. Style — when to ask vs. when to assume`

Two subsections:

- **Fine to assume + state**: judgment calls the cuttlefish can
  make without checking back. Examples: variable naming, exact
  Tailwind classes, sub-component extraction decisions.
- **Stop and ask**: situations that warrant pausing for owner /
  nautilus input. Examples: API surface conflicts, ambiguous
  requirements, scope expansion beyond brief.

This section prevents both extremes — paralyzed cuttlefish that
ask everything, and runaway cuttlefish that decide everything.

### `## 7. When you're done`

- Handoff doc location and required sections (cross-reference
  HANDOFF-TEMPLATE.md)
- Manual validation steps for the owner (numbered list of
  things to check in dev)
- BACKLOG move guidance (this dispatch → Done)
- "Do NOT commit or modify git history" reminder

### `## Final note`

Optional. Single-paragraph closing thought capturing whatever
the brief most wants the cuttlefish to internalize. Often the
"if you find yourself X, stop and re-read Y" warnings.

---

## 3. Tone and style

- **Terse and factual.** This isn't marketing copy. Short
  sentences; bullet lists over prose where they fit.
- **Authoritative without being prescriptive.** The brief tells
  the cuttlefish what's required; leaves the how to their
  judgment where reasonable. The §6 split (assume vs. ask) is
  the explicit boundary.
- **No emoji** other than acceptance-criteria status markers in
  the handoff. Project convention is no decorative emoji.
- **80-column hard wrap** for prose. Code blocks, tables, and
  long URLs exempt.
- **Target length**: 200-500 lines for M dispatches, 100-250 for
  S, 50-150 for XS. If you're way over, you're padding or the
  dispatch should be split. If you're way under, you're skipping
  context the cuttlefish will miss.

---

## 4. Brief-authoring antipatterns

Recorded patterns to avoid (cross-reference WORKING-MODEL.md §6
for the broader antipatterns catalogue):

- **Hand-walking an external UI**: per §1a, describe requirements
  not click paths. Click paths age out within weeks.
- **Over-engineering at brief time**: the temptation is to
  "design completely." Surface the simplest version first; ask
  the owner if it's enough before elaborating. (Empirical: an
  early Route7 dispatch ended up 5x its needed scope because
  the brief speculatively added per-field staging when
  session-level cancel was all the owner wanted.)
- **Ambiguous wording in implementation guidance**: imperative
  language gets translated literally. "Trim whitespace" becomes
  `.trim()` on every keystroke; "use lite SDK" becomes a blanket
  prohibition even where lite is fine. Precision matters where
  the implementer's literal reading would diverge from intent.
- **Internal contradictions**: AC and body must agree.
  Cross-check before shipping. (Empirical: a Route7 brief
  changed §3 styling guidance but missed updating the matching
  AC; implementer correctly followed the body and surfaced the
  contradiction in their handoff.)
- **Stale line numbers**: brief cites `EditorPage.tsx:198`; line
  shifted after a recent dispatch. Verify line numbers before
  shipping the brief; pre-read agent catches these but
  re-running grep yourself is cheap.

---

## 5. Pre-read before dispatch

Per WORKING-MODEL.md §3, every M+ brief gets a pre-read by a
reviewer cuttlefish before the implementer cuttlefish runs.
S-sized is judgment-call; lean toward pre-read on anything that
touches files the nautilus hasn't recently read.

The reviewer cuttlefish:

- Reads the brief in full
- Reads the supporting code the brief references
- Confirms line numbers, file paths, API surfaces, baseline test
  counts, etc. are accurate
- Flags blockers (would cause implementer to fail), should-fixes
  (clarifications), nits (typos / line width), and confirmed-OK
  (assertions verified)

Nautilus folds findings into the brief before spawning the
implementer. Empirically, pre-read catches at least one real
blocker on every M+ dispatch where it runs. The cost-to-value
ratio is heavily favorable.

---

## 6. When this template is NOT the right shape

- **XS dispatches** (~10-30 lines of work) often skip the formal
  brief — the nautilus edits directly. If you find yourself
  writing a brief for something that takes 5 minutes to do, ask
  whether the brief is overhead.
- **Ops dispatches** (no code; runbook only) follow a different
  shape — see flog's
  `dispatch/runbooks/gcp-firebase-env-setup.md` for a worked
  example. Requirements + dated breadcrumbs + rakes-catalogue;
  no AC-driven gates the same way.
- **Multi-dispatch arcs** (XL work) get a design doc first, then
  individual dispatches that reference it. The design doc isn't
  itself a brief; it's a settled-decisions reference.

---

## 7. References

- `HANDOFF-TEMPLATE.md` — the matching response to a brief
- `WORKING-MODEL.md` — lifecycle, pre-read, antipatterns,
  stall-recovery, post-ship-fix protocols
- `BACKLOG-TEMPLATE.md` — size tags + horizon definitions used
  in briefs
- `CUTTLEFISH-NAUTILUS.md` — the conceptual frame for why this
  brief-vs-handoff split works

---

End of brief template.
