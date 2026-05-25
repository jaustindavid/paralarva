# [PROJECT_NAME] — Product Requirements Document (v1)

_Copyright © {YEAR} {OWNER_NAME}. All rights reserved._

> [PROJECT_NAME] is built with Claude (Anthropic) as a continuous
> collaborator. The PRD, ARCHITECTURE doc, and most code are
> produced via human-AI pairing — the planning docs are written
> dense and self-contained so a fresh Claude session can cold-read
> and contribute immediately.

This is the template structure for a project PRD. Each section's
purpose is documented below; the new nautilus fills in the actual
content during the owner interview (see paralarva README §step 2-3).

The shape is proven across multiple projects — keep the sections,
adjust the depth per project. Cold-readable is the test: a fresh
Claude session should be able to read this and act.

---

## 1. Overview

**Purpose**: one-paragraph product description. What is this thing?
Who is it for? What does it do that's distinct?

Sub-sections worth including:

### 1.1 Goals

What the product is trying to achieve. Concrete, measurable when
possible. 3-5 bullets max.

### 1.2 Non-goals

What the product is explicitly NOT trying to do. Often the most
important section — surfaces brand/scope discipline that keeps the
product focused. Helps future-you say "no" without re-deriving the
reasons. If a future BACKLOG item would violate one of these, it
gets a ⚠️ flag pointing back here.

### 1.3 Future-phase / deferred

What might happen later but isn't v1. Points at BACKLOG.md as the
working list.

### 1.4 Philosophical commitments

Anything load-bearing about how the product behaves that doesn't
fit elsewhere. Examples from prior projects:

- "We never collect behavioral telemetry"
- "Sharing is via URL only; no notification system"
- "Data is yours; export and delete are first-class affordances"

These become real constraints on every future design decision.

---

## 2. Target user

Who uses this product? Be specific. "Caterham/Lotus 7 enthusiasts
who plan weekend drives" is more useful than "drivers."

Sub-sections:

- **Primary user(s)** — the canonical user the product is built for
- **Secondary users** (if any) — adjacent uses that the design
  should not break
- **Non-users** — people the product is explicitly NOT trying to
  serve (helps clarify scope; non-goals from §1 often have a
  non-user analog here)

---

## 3. Glossary

Short list of terms and what they mean in this project. Important
when the project introduces vocabulary (e.g., "fork" vs "copy" in
Route7, "trip" vs "session" in a tracker, etc.). Saves
conversation cycles later when the same word means subtly different
things to different readers.

---

## 4. Architecture (high-level)

The shape of the system, not the implementation details. Real
ARCHITECTURE.md is its own doc; this section is the elevator
pitch. Examples of what belongs:

- Client/server topology (SPA? Server-rendered? Hybrid?)
- Primary data store + how clients talk to it
- Auth model (anonymous? signed-in? federated?)
- Hosting / deploy story (Firebase Hosting? Vercel? Self-hosted?)
- External services (Stripe, Google Maps, etc.)

If/when ARCHITECTURE.md exists, this section becomes a brief
pointer to it.

---

## 5. Data model

The core entities, their fields, their relationships. This shapes
both the database schema and the security/access-rules model.

For each entity:

- **Name** (singular, capitalized)
- **Fields** (with types and whether required)
- **Relationships** to other entities
- **Lifecycle** (created when? destroyed when? mutated how?)
- **Ownership** (who can read? who can write?)

This section often grows the most during dispatches — keep it
honest as schema evolves.

---

## 6. Access control

Who can do what. Maps to security rules (Firestore rules, API
permissions, etc.) and to UI gating.

For each entity from §5, document:

- Read permissions
- Write permissions (separate create / update / delete if they
  differ)
- Special carve-outs (e.g., "owner can always delete own data
  regardless of tier")

This section should make rules-writing mechanical: each row is a
rule.

---

## 7. User flows

The primary paths users take through the product. For each flow:

- **Goal**: what is the user trying to accomplish?
- **Steps**: numbered, concrete
- **Acceptance criteria**: how do we know it works?

These flows feed dispatch briefs — when shipping a feature, the
brief's manual-validation steps often map directly to these flows.

3-7 flows is typical for v1. More than that suggests the MVP is
too big.

---

## 8. Cost control

Budget per active user (or per action, or per other relevant unit).
Affects everything downstream: caching strategies, debouncing,
which APIs to use, when to ask for paid tier.

If the product has externally-billed APIs (Google Maps, OpenAI,
etc.), be explicit:

- Expected cost per user-action
- Hard caps that warrant code changes
- Monitoring approach

If the product is cheap-to-free at the planned scale, document
that — it's still useful context for "should we cache?"
conversations.

---

## 9. UI requirements

Visual / interaction conventions that aren't implementation
details:

- Color/typography defaults
- Mobile vs desktop priority (which is primary?)
- Accessibility commitments (WCAG level if any)
- Browser support range
- Performance expectations (first-paint target, etc.)

Light section in most v1s; grows as design decisions accumulate.

---

## 10. v1 Milestones

The MVP scope, broken into dispatches or milestones. This is the
initial roadmap; BACKLOG.md takes over for everything after.

For each milestone:

- Number / name
- Goal (one sentence)
- Acceptance criteria
- Dependencies on prior milestones

This section becomes mostly historical once v1 ships; what's in
flight or upcoming moves to BACKLOG.md's Next / Soon.

---

## 11. Risks & open questions

### 11.1 Risks

Things that could derail the project. Be honest. Examples:

- Dependency on a third-party that could disappear
- Privacy posture that might not survive contact with users
- Cost assumptions that haven't been validated at scale

### 11.2 Open questions for the owner (not blocking)

Things the design conversation hasn't settled yet but doesn't need
to block v1. Capture them here so they don't get forgotten; settle
each via design-conversation-then-update-PRD as they come up.

---

## 12. Implementation guidance for coding agents

Cross-references to AGENTS.md, but also project-specific guidance
worth foregrounding for any agent. Examples:

- "Read PRD §5 and §6 before writing any data-access code"
- "Cost spec in §8 is non-negotiable; the 400ms debounce
  enforces it"
- "Open questions in §11.2 are NOT to be guessed at — ask"

Short section; AGENTS.md does the heavier lifting.

---

## 13. Sustainability philosophy

How the project intends to survive long-term. Two sub-sections
that have proved useful:

### 13.1 Cost posture

How the project keeps operating costs low (or how it handles cost
growth). Includes commitments like "no behavioral tracking" or "no
ads on signed-in surfaces" if those are part of the brand.

### 13.2 Revenue posture (if any)

Whether the project will eventually charge, and if so how. "Free
forever" is a valid posture; "first 100 users free, paid tier
later" is another. Be honest about uncertainty here.

### 13.3 Triggers to revisit

What signals would prompt a re-evaluation of the sustainability
posture. Example: "if monthly Maps API cost exceeds $50, revisit
the rendering strategy for free-tier users."

---

End of PRD template. The new nautilus fills in real content
during the owner interview; this template establishes the
section discipline.

When the project's first PRD ships, this template can be
deleted (the real PRD takes its place). The template is preserved
in the source paralarva folder for next time.
