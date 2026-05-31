# Working model

> This document is part of the **paralarva** bootstrap kit — the
> working patterns used to spin up projects with Claude as
> collaborator. Planning docs in projects derived from this kit are
> written dense and self-contained so a fresh Claude session can
> cold-read and contribute immediately.

This doc is the **operational playbook** for how project work
happens — the patterns evolved across many dispatches on prior
projects, the lessons that produced them, and the boundaries
worth honoring.

It pairs with — but doesn't restate — the conceptual
cuttlefish/nautilus framing (the long-form paper) which
explains *why* this division of labor works. This doc is
about *what we do day-to-day*.

If you're a fresh nautilus or cuttlefish dropped into this
project, read this first. AGENTS.md covers codebase
guardrails; HANDOFF-TEMPLATE.md covers handoff doc shape;
BACKLOG.md is the working list. This doc is the
practitioner's guide that ties them together.

---

## 1. The cast

**Owner** — has the product context. Sets direction,
makes design decisions, reviews + commits. Doesn't write
code directly (with rare exceptions); operates through
delegation.

**Nautilus** (long-context Claude session) — the
architect. Holds the conversation history and the design
intent; carries the design conversation with the owner;
**dispatches cuttlefish for research, design, pre-read,
and implementation, and folds their findings back**;
picks the model tier per dispatch (§5); handles
housekeeping (BACKLOG, post-stall recovery, doc
maintenance). One nautilus per session; sessions may span
hours or days. It coordinates but does not predate — it
provides the shell; the cuttlefish do the work.

**Cuttlefish** (short-context agents) — the
dispatched-agent class. Each is spawned for a single
bounded task, reads what it needs cold (no conversation
history), and is disposed when done. They are **not
downstream-only hands** — the nautilus casts them across
the whole lifecycle, in distinct roles (§3 covers how to
run the upstream ones):

- **Research / exploration** — read-only, fan-out. Survey
  prior art, map unfamiliar code, gather evidence. Return
  *conclusions, not file dumps*; their output feeds
  design. (Fact-finder — verifying an external API/SDK's
  behavior against its docs — is a named flavor of this;
  see §3.)
- **Design** — generate and/or stress-test design options
  (sometimes several in parallel for the nautilus to
  judge), so the owner sees a considered space rather than
  a single guess.
- **Reviewer** (pre-read) — read a brief BEFORE the
  implementer runs it; find blockers / ambiguities / stale
  references. Cheap (~2-10 min); catch issues that cost
  far more in implementer time + recovery.
- **Implementer** — execute a single dispatch end-to-end:
  read the brief cold, run gates, write the handoff,
  update BACKLOG. Optimized for focused execution;
  deliberately scope-constrained.

Same agent class throughout — a research cuttlefish is
not a different creature from an implementer, just a
different prompt and a different bounded task. Cuttlefish
never communicate with each other; every one reports to
the nautilus, which mediates and folds.

---

## 2. The lifecycle

```text
   research / exploration    (cuttlefish fan-out, read-only;
            ↓                  optional — for novel problems)
       fold findings         (nautilus)
            ↓
owner ←→ nautilus            (design conversation; may spawn
            ↓                  design cuttlefish to widen options)
       BACKLOG entry         (decisions captured)
            ↓ (promote to Next)
       brief draft           (nautilus writes;
                              BACKLOG entry is most of it)
            ↓
       pre-read              (reviewer cuttlefish)
            ↓
       fold findings         (nautilus revises brief)
            ↓
       spawn implementer     (cuttlefish runs end-to-end;
                              nautilus picks the model tier, §5)
            ↓
       gates pass            (lint / test / rules / build /
                              markdownlint)
            ↓
       handoff doc           (cuttlefish writes;
                              or nautilus on stall)
            ↓
       BACKLOG move          (Next → Done)
            ↓
       owner review (V2) + commit
```

The arrows are loose. Loops happen — research can reopen a
design question, pre-read findings might trigger a design
re-conversation, the owner might push back on a brief and
we restart, the implementer might flag a stop-and-ask and
kick back to the nautilus.

What's load-bearing:

- **Research and design can be cuttlefish-fed**, not just
  owner+nautilus chat. For a novel problem the nautilus
  spawns research/exploration cuttlefish first and folds
  their conclusions into the design conversation; for a
  wide solution space, design cuttlefish surface options.
  The owner still makes the call — the cuttlefish widen
  and sharpen it (§3).
- **Design conversation lives in the BACKLOG entry**,
  not just in chat. The entry accumulates settled
  decisions and is what the next session (or the brief
  draft) cold-reads.
- **Brief is for the implementer, not the owner**. Owner
  has seen the design conversation; implementer hasn't.
  Brief restates everything the implementer needs from
  cold.
- **Handoff is for the next nautilus + the owner**, not
  for the implementer that wrote it. It's a persistent
  record after the implementer finishes.
- **Owner review (V2) is a real step.** The owner
  validates hands-on after the handoff, at/before commit;
  agents never commit (§5).

---

## 3. Dispatching cuttlefish (research, design, pre-read)

Before — and around — the implementer dispatch, the
nautilus casts cuttlefish in three upstream roles:
**research** (incl. its fact-finder flavor), **design**,
and **pre-read**. All share one discipline: a
purpose-built prompt, a bounded task, a defined report
format, and *read-only* unless the role is implementation.
The nautilus **folds** their conclusions; it never
forwards one cuttlefish's raw output to another.

**Owner cost model — why these are worth invoking
aggressively**: from the owner's perspective the read-only
cuttlefish (research, fact-finder, design, pre-read,
re-read) are effectively free — they spawn, run, and
produce reports without consuming owner attention. The
expensive failures are at the implementer stage, and there
are TWO: implementer **stalls** (obvious; needs triage +
recovery) AND implementer **ships bugs that escape**
(subtler; each becomes a future debug+fix dispatch). Both
share a root cause — inadequate pre-flight preparation.
The upstream cuttlefish exist to protect the implementer
from both. Practical implication: spawn read-only agents
aggressively; don't deliberate about whether to invoke.
Multiple passes on the same brief are fine. The cost of a
pass that finds nothing is small; the cost of a missed
gotcha that stalls (or ships a defect) is much larger.

The subsections below cover each role — **Design** and the
two research flavors (**Fact-finder** and **Research with
a verdict**) are *upstream*, feeding the design
conversation and the brief; **Pre-read** is the *gate* run
just before the implementer. They're grouped here because
they share the dispatch discipline above, so the headings
are ordered for reference, not in strict lifecycle
sequence.

### Design

For a wide or contested solution space, spawn design
cuttlefish to generate and/or stress-test options —
sometimes several in parallel, which the nautilus judges
and synthesizes — so the owner sees a considered space,
not the nautilus's first guess. The owner still decides;
the design cuttlefish widen the menu and surface
trade-offs. For a narrow space, the owner ←→ nautilus
conversation is enough; don't manufacture options for
their own sake (see the over-engineering antipattern in
§6). The "research with a verdict" pattern below is the
heaviest form of design support — a full options-analysis
memo ending in a SHIP / DEFER / DROP call.

### Pre-read

The pre-read pattern: before spawning the implementer
cuttlefish, spawn a **reviewer cuttlefish** with a prompt
asking it to read the brief + the relevant supporting
code, and report any blockers, should-fixes, or
confirmed-OK assertions.

**Why it works**: the nautilus is too close to the brief
it just wrote. A fresh reader catches:

- File paths that don't match reality
- Line numbers that drifted between sessions
- Functions the brief assumes exist but don't (or live in
  a different file than referenced)
- API surface assumptions that need verification
- Numeric baselines that have shifted (test counts, bundle
  sizes)
- Internal contradictions across brief sections
- Missing edge cases that the design conversation hadn't
  considered

**When to pre-read**: every dispatch M-sized and up. XS/S
is a judgment call — if the brief touches a file the
nautilus hasn't read recently, pre-read. If the brief is
genuinely pure mechanical work the nautilus has full
context on, can skip — but we've found that "I have full
context" is usually wrong, and pre-read pays off even on
S-sized dispatches.

**When NOT to pre-read**: ops-only dispatches with no
code (e.g., the OAuth publish runbook). The reviewer has
no privileged access to GCP Console; can't verify the
runbook steps against reality. Pre-read adds no value.

**How to spawn**: nautilus uses the Agent tool with a
purpose-built reviewer prompt. The prompt names the
brief, the supporting files to read, the specific checks
to run, and the report format (BLOCKING / SHOULD-FIX /
NITS / CONFIRMED-OK). Reviewer is explicitly told NOT to
modify any files.

**Real evidence**: across the recent dispatch arc,
pre-read has caught at least one real blocker on EVERY
dispatch where it ran. Examples:

- `reduceDraft.ts` didn't exist — reducer is inside
  `draftState.ts`. The brief referenced the wrong file 11
  times. Implementer would have failed cold.
- `getCountFromServer` is not in `firebase/firestore/lite`.
  The nautilus's pre-flight asserted 100% compatibility;
  pre-read found the one exception. Would have caused a
  TypeScript compile failure.
- A `saveRoute.ts` projection silently dropped two new
  fields. TypeScript wouldn't catch (fields were
  optional); pre-read spotted it by tracing data flow.

The cost-to-value ratio of pre-read is heavily favorable.
Skip with caution.

**Re-read after substantive fixes.** When pre-read finds
three or more blocking issues and the nautilus folds them
in, **spawn a second pre-read pass** before handing to the
implementer. The fix-cycle introduces its own drift —
prose updates without matching AC updates, new
contradictions, accidentally re-introduced bugs. The
second pass is cheap (a focused "verify each fix landed
cleanly + no new issues introduced" prompt) and reliably
catches AC-vs-prose drift that a single-pass review
misses. Heuristic threshold: 3 blocking issues warrants
re-read; fewer is judgment-call. This pattern has caught
real residual drift on every dispatch where it's run.

### Fact-finder — verifying external API assumptions (a research flavor)

The fact-finder pattern: before drafting a brief that
relies on specific behavior of an external API or SDK,
spawn a **read-only research cuttlefish** to verify those
behaviors against the actual documentation (and, when
docs are ambiguous, against a small scratch probe). The
output is a fact-sheet that informs the brief; the brief
then carries the verified-and-incorporated findings into
the implementer's hands.

**Why this is separate from pre-read**: pre-read
verifies internal consistency — does the brief match the
codebase? Fact-finder verifies external assumptions —
does the API actually work the way the brief assumes?
The two are complementary; both can run on the same
dispatch.

**When to invoke**:

- Any dispatch touching an external API or SDK that
  hasn't been recently exercised (Google Maps Platform
  variants, Firebase products newly reached for, third-
  party libraries)
- Any time the brief says "the SDK supports X" or
  "Google's API does Y" without recent verification
- Especially: APIs with known quirks (Maps Platform's
  legacy-vs-new variants, anything with documented but
  rarely-exercised behaviors like InfoWindow lifecycle
  semantics or `next_page_token` requirements)

**When NOT to invoke**:

- Pure internal work (reducer changes, UI in existing
  patterns, persistence-shape changes that don't touch
  external APIs)
- Dispatches where the API surface has been exercised
  by a recent dispatch the nautilus has full context on

**How to spawn**: similar shape to pre-read. The prompt
names the brief, names the specific API behaviors to
verify, asks for a fact-sheet with explicit "assumption
X is correct/incorrect" callouts. Reviewer is explicitly
told NOT to write code; output is documentation reading +
synthesis only.

(The owner-cost-model rationale for spawning these
aggressively is in the §3 intro above and applies to
fact-finder in full.)

**Real evidence**:

- Google Places API InfoWindow doesn't auto-close other
  InfoWindows; the natural implementation shape (one
  InfoWindow per marker) would have been wrong. Fact-
  finder caught this from the docs before the brief was
  written.
- Google Places API legacy JS library doesn't expose
  `next_page_token` as a string (opaque pagination
  object); a brief that designed cross-session
  pagination around a stored string would have stalled
  the implementer in a self-doubt loop. (This case was
  surfaced reactively after a stall, not preemptively;
  the fact-finder pattern was filed specifically because
  this miss was expensive.)
- Tailwind utility classes don't reliably resolve inside
  Google's managed InfoWindow DOM context; inline
  `style` attributes are required. Fact-finder caught
  this; brief specified inline-style from the start;
  implementer shipped clean.

**Recommended pre-flight pipeline** (each step optional
based on dispatch shape):

```
[design pass: internal integration]      ← if new feature
                                            with significant
                                            code-shape questions
       ↓
[fact-finder: external API behavior]     ← if external API
                                            surface is touched
       ↓
draft brief (nautilus)
       ↓
[pre-read: brief consistency + facts]    ← effectively always
       ↓
fold fixes (nautilus)
       ↓
[re-read: verify fixes + check drift]    ← if pre-read found
                                            3+ blocking issues
       ↓
implementer
```

For pure-internal small dispatches: pre-read alone is
usually sufficient. For external-API dispatches: at
minimum fact-finder + pre-read. For complex new features
spanning multiple integration points: design pass first,
then both.

### Research with a verdict — running an open design question to a call

The research-with-a-verdict pattern: spawn a read-only agent on a
design question the BACKLOG entry hasn't settled, ask for a memo
that explores the options and ends with an explicit verdict.
Used to evaluate features whose sizing, shape, or product
justification carry real uncertainty — the kind of question
where "design it, then decide not to do it" would have wasted
implementer time.

**How this differs from the fact-finder flavor above**:
fact-finder verifies external API behavior (does the SDK do X?).
This pattern explores design questions (should we do X at all, and
if so what shape?). Different output: fact-finder produces a
fact-sheet that informs a brief; this produces a verdict that
informs a horizon move (Soon→Later, sizing correction, scope
split). Both can run on the same dispatch but serve different
needs.

**When to invoke**:

- A BACKLOG item where you suspect the sizing is wrong (XL might
  actually be S; S might actually be M)
- A feature where the owner has stated reservations ("personally
  not convinced") and a focused pass could either validate the
  reservation or surface evidence against it
- A speculative item that's been on Later long enough to have
  evolved (the original framing may be stale)
- Any item where a credible negative verdict (DEFER / DROP)
  would itself be a useful outcome, not a "wasted" pass

**When NOT to invoke**:

- Items with clear scope that just need to be promoted when
  their trigger fires (no design questions to wrestle with)
- Items already settled by an earlier design conversation
  (status `[~]` design captured)
- PRD-conflict items where the question is product philosophy
  rather than implementation design (research doesn't help with
  "should we revise the PRD")

**Verdict vocabulary** (use these labels exactly):

- `SHIP` — ship now; design is settled enough
- `SHIP-WITH-MODIFICATIONS` — ship, but the original framing was
  wrong; here's the corrected shape (e.g., "ship Half B; defer
  Half A")
- `DEFER` — don't ship yet; named trigger conditions for when to
  re-evaluate
- `DROP` — don't ship on any visible horizon; lighter
  alternatives are sufficient
- `NEEDS-PRD-REVISION-FIRST` — can't even answer until the PRD's
  governing constraint is revisited (rare; reserve for actual
  PRD conflicts)

Standard labels matter because they let the orchestrator (and
owner) parse the result instantly without reading the memo body.
"DEFER with 4 triggers" lands faster than a paragraph of prose.

**Trigger-condition convention**: every DEFER or
DROP-of-the-full-shape verdict should name explicit trigger
conditions for revisiting. The recurring pattern across the
passes that established this section: "two-of-the-following
independently" or "two testers independently request X, with two
being a hard threshold (one is interesting signal, not action)."
Sharpening vague "real evidence" promote-when conditions into
countable triggers is one of the highest-value research-
cuttlefish side-effects.

**The "split a request into halves" move**: when an item bundles
multiple separable concerns, instruct the agent to evaluate each
independently. Concrete example: the search-along-route memo
split the tester's request into Half A (search across whole
polyline — expensive, speculative) and Half B (one-click add-as-
waypoint on existing results — cheap, evidence-grounded),
producing a SHIP-WITH-MODIFICATIONS verdict that recommended
Half B alone. A monolithic "is this feature worth shipping?"
verdict would have lost that nuance. Bake this prompt guidance
into research prompts where the BACKLOG entry conflates multiple
concerns.

**Sizing-correction is a legitimate outcome**. Three of four
passes on 2026-05-27 returned with "the BACKLOG entry's sizing
is overstated; the honest size is X." When this happens, fold
the corrected sizing into the BACKLOG entry as part of the
verdict-incorporation. This is curation, not scope creep.

**BACKLOG annotation conventions** (three states of the same
pattern):

- **In-flight**: `_Research pass in flight (spawned YYYY-MM-DD);
  memo target: dispatch/<topic>-research.md._` Appended to the
  entry preamble when the agent is spawned. Documents that
  research is happening without blocking other planning.
- **Completed**: `Full research memo:
  dispatch/<topic>-research.md` becomes a clause in the entry's
  provenance bracket. The persistent reference.
- **Opportunity** (not yet spawned): `_Research opportunity:
  <one-line reasoning>._` Inline annotation on items where the
  pattern would apply but the owner hasn't authorized the spawn
  yet. Gate criterion: "would this item benefit from a focused
  research pass independently of waiting for the trigger
  conditions to fire?" If yes, annotate. If no (clear scope,
  design already settled, trigger-gated correctly), don't.

**How to spawn** (similar shape to pre-read / fact-finder):
purpose-built prompt naming the BACKLOG entry, supporting files
to read, the memo structure to produce (problem-restatement →
options-analysis → trade-offs → cost projection → interaction
with adjacent features → recommendation → dispatch sketch if
SHIP). Explicit invitation in the prompt to come back DEFER or
DROP — "research that fails to deliver a negative verdict when
one is warranted wastes the owner's time."

**Parallel spawning works**: three research passes in flight at
once on 2026-05-27 returned cleanly with no coordination issues.
A round of three is roughly the right size; more starts to
fragment owner attention on result review.

**Real evidence** (the 2026-05-27 arc that established this
section as load-bearing):

- Viewport search: `DEFER` (cost not the blocker; product
  justification is)
- Search-along-route: `SHIP-WITH-MODIFICATIONS` (split into
  halves; ship Half B, defer Half A)
- Trip grouping: `DROP` the full data model (URL composition
  shipped earlier the same day is sufficient; cheap middle
  path named for future trigger response)
- Discovery feed: `DEFER` with re-sizing from `[XL — milestone]`
  to `[S — curated only]` (XL framing reflected a maximal shape
  the PRD wouldn't allow anyway)

Three of four returned with sizing corrections. Two of four
identified middle-path alternatives the BACKLOG hadn't named.
Each pass cost roughly ~$1 of API spend and ~5-15 minutes of
agent runtime; aggregate avoided ~3 design-then-reject dispatch
cycles. The cost-to-value ratio mirrors fact-finder's: heavily
favorable. Spawn aggressively.

---

## 4. The fresh-head principle

Every markdown deliverable in projects derived from this
kit opens with the same preamble (project-name and
copyright year substituted in):

> [PROJECT_NAME] is built with Claude (Anthropic) as a
> continuous collaborator. The PRD, ARCHITECTURE doc, and
> most code are produced via human-AI pairing — the
> planning docs are written dense and self-contained so a
> fresh Claude session can cold-read and contribute
> immediately.

This is the operating norm, not boilerplate. The test
for any doc — brief, handoff, BACKLOG entry, even this
one — is: *if a fresh Claude session with no prior
context reads this, do they have enough to act?*

Practical consequences:

- **No conversational shorthand** in docs. "As we
  discussed" → name the decision and link the source.
- **Every brief restates context** the implementer needs,
  even if the nautilus thinks it's obvious.
- **Handoffs document what shipped** including the
  reasoning behind judgment calls, not just the diff.
- **BACKLOG entries grow with the conversation** — an
  item promoted from Later to Soon often gains 30-100
  lines of captured decisions before it's promoted to
  Next.

The owner can also be the "fresh head" — coming back
to a project after a week off, the docs should let them
reconstruct what was happening without trawling chat
history.

---

## 5. Operational conventions

Patterns that have emerged across dispatches and now
function as defaults. Deviation requires a reason; the
reason goes in the handoff.

**This section stays small and intentional.** Growth here
means adding a new universal rule applied to every
dispatch, which is rare. The right home for "lessons
learned" is §6 Antipatterns, which are pattern-matched
case-by-case rather than always-applied. §5 is reserved
for "we always do X" rules where the cost of NOT doing
X is large enough to justify zero context-sensitivity —
gates, no agent commits, handoff-before-polish.
Promoting an antipattern up to §5 should require real
evidence the rule earns its keep on every dispatch shape,
not just the one that produced the lesson.

**The BACKLOG entry IS most of the brief.** Items in
Later are sparse. As design conversations happen, the
entry accumulates decisions. By the time something is in
Next, the entry has settled most of the questions; the
brief mostly translates this into dispatch format
(required reading, ACs, gates, handoff guidance).

**AC prefix numbering** (e.g., S1, R2, T3). Each
subsection numbers independently from 1; the handoff
references ACs by prefix. Avoids MD029 ordered-list
warnings and makes status easy to scan
("S1-3 ✅, R1-5 ✅, T1-2 ✅").

**"Files in play" + "Files NOT to touch"** as explicit
lists in every brief. The latter is a guardrail; the
former is a budget. Implementer flags any deviation in
the handoff.

**"Stop and ask" section** in every brief. Explicit list
of situations that warrant pausing rather than guessing.
Examples: "If the rule helper conflicts with an existing
helper of the same name, stop and ask." "If Firestore
demands a composite index during testing, propose the
addition before adding it." Reduces failed-execution
costs by giving the implementer permission to surface
ambiguity.

**"When to swap back" tripwires** for reversible
non-trivial decisions. E.g., the Firebase tree-shaking
dispatch captures the conditions under which the
firestore/lite swap would need reverting (real-time
listener need, offline persistence need, etc.). The
tripwire stays in the BACKLOG entry's Done version
forward — future contributors see when a decision
should be revisited.

**Model tier fits the risk.** The nautilus picks each
cuttlefish's model tier per dispatch: a **cheaper /
faster tier** for mechanical, low-risk work (UI, pure-
function refactors, doc edits, mechanical test updates)
and a **stronger tier** for anything where a subtle wrong
call is expensive — security, data-access rules, schema /
data-model changes, money-touching logic — and for the
pre-reads of those same surfaces. Make it an explicit
decision (state the chosen tier in the brief's model
line), not a silent default. When unsure, default up: the
cost delta of the stronger tier is trivial next to one
escaped bug on a sensitive surface.

**Gates are non-negotiable.** Lint, markdownlint, unit
tests, rules tests, and build all pass before the handoff
is written (the exact gate commands are project-specific —
see the project's AGENTS.md / package scripts). If a gate
fails, fix it; don't paper over.

**Handoff before polish.** As soon as gates pass,
implementer writes the handoff — before doing any
"while I'm here" cleanup. This survives stalls (see §7).
Then do the BACKLOG move as the final step.

**No git commits by agents.** Nautilus and cuttlefish
never commit. Owner reviews and commits, period. The
git tree is the owner's source of truth.

**Owner review (V2) is a named terminal step.** After the
handoff and BACKLOG move, the work is "done" only in the
agent sense — V1, validated by gates. The owner's hands-on
review (V2) is a distinct step in the lifecycle (§2), not
an afterthought: the owner runs the manual-smoke steps the
handoff lists, exercises the real artifact, and only then
commits. Bugs that gates can't catch — layout, interaction
feel, real-data edge cases — surface here. Treat V2
findings as first-class: an XS one folds inline + appends
to the handoff (§8); larger ones become their own
dispatch.

---

## 6. Antipatterns we've learned to avoid

These are real lessons from real dispatches. Some of
them pull in opposite directions; that's noted
explicitly. The art is in the synthesis.

**§6 accumulates freely.** Each entry is a pattern to
recognize, not a rule to follow. The brief-writer
pattern-matches against §6 entries on a per-dispatch
basis: "is my current brief vulnerable to this specific
failure mode?" If yes, apply the named mitigation. If no,
ignore. **Do NOT treat §6 entries as a universal
checklist** — that would ironically reinforce the
over-engineering antipattern (§6.1), inflate briefs with
defensive language unrelated to the dispatch at hand, and
dilute implementer attention. The descriptive form
("concrete example + when the pattern applies + soft
mitigation") is deliberate; antipattern entries that drift
toward prescriptive "every brief MUST" framing belong in
§5 instead, and §5 should resist them unless they earn
their keep universally.

### 6.1 Over-engineering at brief time

The nautilus's instinct is to "design completely." This
sometimes produces a spec that's 5x what the owner
actually wants.

**Concrete example**: the Stage-then-commit dispatch
was originally designed as per-field staging
(WaypointList + MapEditor + parallel staging state +
~150 lines + 4 design questions). Owner's actual want
was a single session-level Cancel button next to Save
(~30 lines, 1 question). The over-engineered version
got drafted in detail before the owner read it and
said "this is way more complex than what I had in
mind."

**Mitigation**: when drafting a design conversation,
surface the simplest version first and explicitly ask
"is this enough?" before elaborating. The owner is the
backstop against over-spec; nautilus's job is to
present the cheap option, not assume the expensive
one is needed.

### 6.2 Ambiguous wording in briefs

Imperative-sounding language in a brief gets translated
literally by the implementer. Subtlety in the intent
gets lost.

**Concrete example**: the Waypoint notes/labels brief
said "Handler trims whitespace; empty string → null."
Intent was "if the value is effectively empty,
normalize to null." Implementer (correctly) translated
to `.trim()` on every dispatch, which stripped trailing
whitespace on every keystroke, making it impossible to
type spaces. Owner reported within minutes of shipping.

**Mitigation**: when an instruction has subtlety, say so
explicitly. "Trim only for emptiness check, store raw
value" is more precise than "trim whitespace." If a
brief's wording could be read two ways, the implementer
will pick the one the brief literally said.

### 6.3 The intentional tension between 6.1 and 6.2

These two pull in opposite directions:

- **6.1** says don't over-specify; leave latitude.
- **6.2** says don't under-specify; be unambiguous.

This conflict is real and unresolvable in the abstract.
The synthesis is **about WHERE precision matters**:

| Precision matters | Latitude is fine |
|---|---|
| Intent ("what is this trying to achieve") | Implementation details ("which loop construct") |
| Invariants ("X must hold after Y") | Component extraction ("inline vs. helper file") |
| Boundaries ("in scope / out of scope") | Variable naming, Tailwind class choice |
| Numeric thresholds chosen by design | Performance micro-optimizations |
| Naming of contracts (action names, props) | Comment wording |
| Field shape decisions (optional vs. required) | Test framework idiom (`it` vs. `test`) |

When a brief is precise about intent but loose about
implementation, the implementer can use judgment on the
"how" while the "what" is unambiguous. When a brief is
loose about intent, the implementer guesses — and the
guess is often wrong.

**Practical heuristic**: re-read the brief asking "if
the implementer reads each sentence the wrong way, does
the result still work?" If no, tighten that sentence.
If yes, you're done.

### 6.4 Skipping pre-read on "obviously simple" dispatches

Early in the working pattern, we'd skip pre-read on
S-sized work, reasoning that the cost outweighed the
value. We've stopped doing this. Pre-read has caught a
real blocker on every dispatch where it ran. The
cost-to-recovery ratio is much higher than the
cost-to-pre-read.

**Exception**: ops-only dispatches where the reviewer
has no privileged access (OAuth publish runbook,
deployment runbooks). Pre-read there is performative.

### 6.5 Briefs that hand-walk an external UI

Any brief that prescribes click-by-click paths through
a vendor console (GCP Console, Firebase Console, AWS
Console, Stripe Dashboard, etc.) ages out quickly.
External UIs reorganize constantly — a single execution
session can encounter 5+ UI drifts.

**The pattern**: describe **requirements** (stable: what
state the system must be in after this step), optionally
include a **dated breadcrumb** (current UI path, marked
with the date last verified), and trust the executor to
navigate. The brief sets the destination; the executor
finds the current controls through whatever the UI is
labeled as today.

**Empirical**: flog's first ops dispatch (M1
infrastructure) hit 7+ UI reorganizations during one
afternoon — OAuth consent wizard condensed, Test Users
moved under an Audience tab, Authorized Domains moved
to a Branding sub-screen, validation tightened,
Firebase Auth panel moved from Build to Security,
Firestore moved to Databases, Hosting moved to Hosting
& Serverless. A click-path brief would have stalled at
each drift; the requirement-shaped sections sailed
through.

See `BRIEF-TEMPLATE.md` §1a for the canonical form. The
two-section pattern (Requirement + dated Current path)
makes future updates "refresh the breadcrumb," not
"rewrite from scratch."

### 6.6 Implementer stream stalls

Periodically, an implementer cuttlefish's stream
times out mid-execution. Happened on Dispatch 1 (Admin
allowlist gate) and the Editor session-cancel
dispatch. Pattern: the code is complete; gates pass; the
stream stalls before the handoff doc is written.

This isn't a defect to fix; it's a property of the
system to plan around. See §7 for the recovery
protocol.

The instruction "write the handoff immediately when
gates pass" is the primary mitigation. If the
implementer finishes after writing the handoff, the most
important artifact survives. If it stalls before, the
nautilus has to reconstruct.

### 6.7 Conditional-render breaks sibling layout assumptions

When a brief makes an element conditional ("render X only
when feature Y is active"), sibling elements may have
inherited layout responsibilities from X that disappear
along with X. The conditional render then exposes a
regression in the off-state that no snippet in the brief
illustrated.

**Concrete example**: the multi-route-overview dispatch
moved the home-row left-padding from the `<Link>` (where
it originally lived) onto a new checkbox `<label>` sibling
via `pl-4 pr-2`. The follow-up combine-mode dispatch made
the checkbox conditional on `combineMode === true` — and
the label's `pl-4` went conditional with it. When the user
was out of combine-mode, route titles sat flush against
the viewport edge. Owner caught it during smoke; XS fix
on the Link's className with a `${combineMode ? '' :
'pl-4'}` clause.

**Mitigation at brief stage**: when specifying a
conditional render, the brief should explicitly name the
pre-conditional layout responsibilities of the affected
element. One extra sentence — "the checkbox label
currently provides the row's left padding; the Link must
take over in the off-state" — would have made the gap
obvious to the implementer and to pre-read.

**Mitigation at pre-read stage**: when reviewing a brief
that makes an existing element conditional, explicitly
model both the ON-state and the OFF-state of the
surrounding layout. The brief's snippets always show the
on-state (that's where the new behavior is); the
regression lives in the off-state that no snippet
illustrates. "What does the row look like when
`combineMode` is false?" is the question pre-read should
ask but didn't, on this dispatch.

### 6.8 Library-pattern lift carries hidden invariants

When lifting a working pattern from one surface to
another — same library, same configuration, even the same
code shape — the new surface's children may violate
invariants the original silently depended on. The lifted
pattern compiles, looks identical to its source, and
breaks at runtime in a way that file-level review misses.

**Concrete example**: WaypointList established Route7's
canonical dnd-kit sortable pattern — spread `{...attributes}
{...listeners}` on the whole row; `MouseSensor` activation
distance of 8px prevents accidental drag-starts on
intended clicks. The pattern works on WaypointList because
its row children are inputs / textareas / small buttons,
all of which natively self-handle pointer events. The
OverviewPage legend lifted the same pattern verbatim — but
its row contains a `<Link>`. When the user drops a drag,
the mouseup bubbles to the Link as a click, navigating to
the route viewer instead of completing the drop. The
`activationConstraint` catches "intended click,
accidentally dragged" but does NOT catch "intended drag,
accidentally clicked at drop" — that's a separate
concern requiring the documented "dedicated drag handle"
pattern. Owner caught it during smoke; fix replaced
whole-row drag with a grip-handle column.

**Mitigation at brief stage**: when a brief instructs
"lift the pattern from existing surface X verbatim," the
brief must enumerate the invariants surface X's children
satisfied — then verify the new surface's children satisfy
the same invariants. For the sortable pattern specifically:
"if the new row contains an `<a>`, `<Link>`, or anything
whose default click behavior is navigation, do NOT use
whole-row drag — use a dedicated drag handle." That
sentence in the original brief would have prevented this
bug entirely.

**Mitigation at pre-read stage**: when reviewing a brief
that lifts a pattern from another surface, explicitly diff
the two surfaces' child elements. If the new surface
introduces an interactive child the original didn't have,
flag the pattern-lift as needing adjustment. This is one
of the few cases where pre-read should look beyond the
literal text of the brief into the actual code shape of
both surfaces involved.

**Generalizable lesson**: "lift this pattern from X" is a
shortcut that hides the constraints X was operating
under. Briefs should name the constraints; reviewers
should verify the new surface satisfies them.

### 6.9 Agent-produced markdown needs lint discipline imposed in the prompt

Every research-cuttlefish memo across the 2026-05-27
arc came back with markdownlint errors — MD004 (rogue
`+` characters at line start interpreted as list
markers), MD032 (lists without surrounding blank
lines), MD034 (bare URLs). Same pattern across three
different agents on three different topics — not a
one-off. Agents trained on lots of GitHub-flavored
markdown produce lint-failing prose by default; many
project markdownlint configs are stricter than
GitHub's permissive render.

**Concrete example**: a research memo's L407 had
`+ ad-hoc URL composition already covers...` — rogue
`+` at line start, interpreted as a list marker by
markdownlint, triggering MD004 and MD032 both. The
same memo introduced a bare `https://example.com` URL
in prose, triggering MD034. Across four memos that
landed in one evening, ~7 lint errors needed
post-hoc nautilus fixing.

**Mitigation in the prompt**: one paragraph added to
the standard research / fact-finder / any-agent-that-
produces-markdown prompt template:

> Run `npx markdownlint-cli2 <your output path>` on
> your memo before exiting; resolve any errors. The
> project's lint config is stricter than GitHub's
> render — common issues are MD004 (avoid `+` at
> line start; use `-` or rewrite), MD032 (lists need
> a blank line before and after), MD034 (wrap bare
> URLs in backticks or write them as `[text](url)`
> links).

Total prompt-cost: one paragraph. Total nautilus-time
saved per research round: ~5 minutes of post-hoc lint
fixing. Pure win.

**Generalizable lesson**: when delegating
documentation output to an agent, the project's lint
discipline must be imposed in the prompt — agents
won't infer it from "this is a Route7 document."
Applies to: research memos, fact-finder fact-sheets,
implementer-cuttlefish handoff docs (already in their
gates list but worth re-checking the prompt
explicitly mentions markdownlint), and any future
agent-produced markdown.

### 6.10 Fixed-position elements collide with app chrome

`position: fixed` is viewport-relative; `sticky`-
positioned app chrome (headers, footers, action bars)
is ALSO viewport-relative. They occupy the same
coordinate space and overlap when offsets are smaller
than the chrome's footprint.

**Concrete example**: the printable-turn-by-turn
dispatch specified `fixed top-4 right-4` on the
Copy-as-text button. The AppHeader is `sticky top-0
z-30` and ~56-88px tall (more on iOS with safe-area-
inset-top). The button sat behind the header until
mobile Safari's URL-bar collapse briefly revealed it
during scroll — the owner caught it during smoke.
Fix: `top-20` (5rem = 80px) to clear the header's
footprint.

**When the pattern applies**: briefs that specify
fixed-positioned UI on pages that render INSIDE an
app-shell layout. Doesn't apply to pages that render
standalone (a future marketing site, a print-only
page that hides the AppHeader, etc.).

**Mitigation at brief stage**: prefer behavioral
specs ("always visible, sits clear of the header")
over coordinate specs ("top-4"). When coordinates are
necessary, name the chrome footprint they need to
clear — "the AppHeader at `sticky top-0` is ~56-88px
tall; this button must offset below that."

**Mitigation at pre-read stage**: if a brief
specifies `fixed` positioning on an app-shell page,
mentally place the element on the page and check for
app-shell overlap. The check is small but reliably
catches a class of layout bugs that visual review
catches only AFTER ship.

**Note (per §6 prefix)**: this entry is descriptive,
not prescriptive. Apply when the dispatch ships
fixed-positioned UI inside an app-shell layout; ignore
when it doesn't. The brief-writer's judgment about
applicability is the load-bearing skill, not rote
checklist execution.

---

## 7. Stall-recovery protocol

When an implementer cuttlefish stalls (stream-watchdog
timeout), the nautilus runs this sequence:

1. **Don't immediately re-spawn.** Check current state
   first — file mod times in the last hour, `git status`
   if available, gate results. The implementer often
   completed the code before stalling.
2. **Run all five gates manually.** If they pass, the
   code is good; only housekeeping is missing.
3. **If gates pass**: write the handoff doc post-hoc as
   the nautilus. Spot-check the actual implementation
   against the brief (read the key files, verify the
   substantive decisions). Note the stall in the handoff
   ("Handoff written by the nautilus after the
   implementer cuttlefish completed the substantive
   code work but stalled before reaching the
   housekeeping steps.").
4. **If gates fail**: triage. If it's a small fix
   (typo, missing import), the nautilus can fix inline
   and proceed to step 3. If it's a structural issue,
   probably re-spawn the implementer with a continuation
   prompt referencing the partial state.
5. **Do the BACKLOG move** as the nautilus.
6. **Owner reviews + commits** as normal.

The handoff written post-stall is the same quality as
one written by the implementer — it just costs the
nautilus's time instead of the cuttlefish's. Owner sees
the same artifacts in the same locations.

---

## 8. Post-ship-fix protocol

After a dispatch ships, the owner sometimes finds a
small issue during validation. The protocol depends on
the size of the issue.

### XS fixes (inline, append to handoff)

If the fix is XS (a few lines, contained to the dispatch's
intended behavior space, doesn't change the brief's
scope), the nautilus fixes inline as a follow-up commit
and appends a "Post-ship fix" section to the original
handoff. No new dispatch.

**Examples we've used this for**:

- Auth record deletion: a small modal-state bug found
  immediately post-ship. One-line fix; appended to the
  handoff.
- Waypoint notes/labels: three XS adjustments shipped
  alongside the dispatch's main commit — spaces bug fix,
  threshold tuning (30m → 100m), trim-on-save. Each
  documented in the handoff.

The owner's framing on this (2026-05-23): "XS stays in
scope of where we started. M+ would justify deferring
to a future dispatch."

### S+ fixes — file a follow-up dispatch instead

If the post-ship issue would require S or larger to
address properly, **don't patch it inline**. File as a
follow-up item in BACKLOG. Reasoning:

- An S+ fix usually implies the original design missed
  something structural. Patching extends the original
  commit beyond its tested scope.
- Bisect signal — keeping each commit focused on one
  intent makes future debugging easier.
- The owner's review attention is targeted at the
  dispatch's scope when they reviewed it; an S+ patch
  appended post-hoc dilutes that review.

We haven't actually hit an S+ post-ship issue yet (the
biggest has been the spaces bug, which was XS), so this
boundary is theoretical. But the principle: if "patch
it real quick" starts to feel large, that's the signal
that "real quick" isn't the right response.

### M+ fixes — definitely a new dispatch

For M+ post-ship issues, the original dispatch's
implementation was working from an incomplete
understanding of the problem. A new dispatch with its
own design conversation + brief + pre-read is the
correct response. The original handoff still notes the
issue ("see follow-up dispatch X") for future
attribution, but the fix itself is its own
self-contained piece of work.

---

## 9. Tools and where they live

- **Briefs**: `dispatch/<name>.md` (per dispatch)
- **Handoffs**: `dispatch/<name>-handoff.md` (per
  dispatch, matched pair with the brief)
- **BACKLOG**: `dispatch/BACKLOG.md` (single file, all
  horizons + Done)
- **This doc**: `dispatch/WORKING-MODEL.md`
- **Handoff structure spec**:
  `dispatch/HANDOFF-TEMPLATE.md`
- **Codebase guardrails**: `AGENTS.md` (project root)
- **Product spec**: `PRD.md` (project root)
- **Architecture**: `ARCHITECTURE.md` (project root)

A typical dispatch leaves behind two files in
`dispatch/` (brief + handoff) plus one BACKLOG edit
(Next → Done with a substantive entry). Nothing else.

---

## 10. What this doc is NOT

- **Not a substitute for the cuttlefish/nautilus paper**,
  which establishes the conceptual frame (why this
  division of labor works, what makes a cuttlefish a
  cuttlefish). This doc is the practitioner's playbook
  that lives in the codebase.
- **Not a codebase guide**. AGENTS.md and ARCHITECTURE.md
  cover the project-specific patterns.
- **Not a template**. HANDOFF-TEMPLATE.md is the
  template. This doc explains the operating context.
- **Not exhaustive**. New patterns emerge; this doc
  updates as they do. Each major addition is itself a
  meta-decision worth a brief design conversation
  between nautilus + owner.

---

## 11. When to update this doc

When a new pattern earns its keep (used 2-3 times,
clearly distinct from existing patterns) — add it. When
an antipattern bites us multiple times — capture it.
When the synthesis of two existing patterns needs
naming — name it.

The maintenance cadence is opportunistic, not scheduled.
Owner can flag "we should update WORKING-MODEL with X"
during any session; nautilus drafts the addition,
owner reviews + commits.

---

End of working model.
