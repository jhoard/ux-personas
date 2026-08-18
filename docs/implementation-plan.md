# UX Personas Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `ux-personas` Claude Code skill: research-grounded persona creation, a portable persona library, and virtual-user prototype testing via isolated persona subagents.

**Architecture:** Five Markdown files form the skill — one harness-specific orchestration file (`SKILL.md`) and four portable knowledge files (schema, virtual-user instructions, report template, enterprise starter). Source of truth lives in this repo at `skills/ux-personas/`, symlinked into `~/.claude/skills/ux-personas` so it is simultaneously git-versioned and live. Personas are plain Markdown files with YAML frontmatter in `~/.claude/personas/` (global) or `<project>/personas/` (per-engagement). Test runs spawn one subagent per persona via the Agent tool; subagents drive prototypes with browser tools and return structured traces; the main session synthesizes a findings report.

**Tech Stack:** Markdown + YAML frontmatter only. No code dependencies. Verification uses `bash` checks and agentic acceptance tests.

**Spec:** `docs/superpowers/specs/2026-08-18-ux-personas-design.md`

## Global Constraints

- Only `SKILL.md` may contain Claude Code-specific instructions (Agent tool, browser tool names). All other files must be harness-agnostic prose.
- Persona `provenance` is exactly one of `research | imported | generated`; `confidence` exactly one of `high | medium | low`; personas with `provenance: generated` are locked to `confidence: low`.
- Simulation-critical frontmatter fields (required on every persona): `tech_fluency`, `patience`, `reading_style`, `domain_knowledge`, `accessibility`.
- Outcome taxonomy is exactly: `completed | gave-up | blocked | incomplete (technical)`. `gave-up` is a finding; `incomplete (technical)` is not.
- The report limitations footer is auto-included and non-removable.
- Fields not supported by research evidence are marked `inferred` inline, never silently guessed.
- Error handling principle: honest failure over improvisation — no fabricated test results, ever.
- Commit messages end with `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.

---

### Task 1: Scaffold + persona schema reference

**Files:**
- Create: `skills/ux-personas/persona-schema.md`
- Create: symlink `~/.claude/skills/ux-personas` → `<repo>/skills/ux-personas`
- Create: directory `~/.claude/personas/`

**Interfaces:**
- Produces: the schema document that `SKILL.md` (Task 5) instructs creation flows to follow, and that acceptance tests (Tasks 7–8) validate personas against. Field names produced here are load-bearing everywhere: `name`, `display_name`, `archetype`, `provenance`, `confidence`, `sources`, `created`, `client`, `tech_fluency`, `patience`, `reading_style`, `domain_knowledge`, `accessibility`, `attributes`.

- [ ] **Step 1: Create directories and symlink**

```bash
mkdir -p /Users/jameshoard/AI/jeh3-incorporated/skills/ux-personas/starters
mkdir -p ~/.claude/personas
ln -sfn /Users/jameshoard/AI/jeh3-incorporated/skills/ux-personas ~/.claude/skills/ux-personas
```

- [ ] **Step 2: Verify the symlink resolves**

Run: `ls -la ~/.claude/skills/ux-personas/ && test -d ~/.claude/skills/ux-personas/starters && echo OK`
Expected: `OK`

- [ ] **Step 3: Write `skills/ux-personas/persona-schema.md`**

````markdown
# Persona Schema

A persona is one Markdown file: YAML frontmatter (machine-readable simulation
fields) plus narrative body sections. Filename is `<name>.md` where `name` is
the frontmatter slug.

## Layer 1 — Universal core (required, domain-agnostic)

```yaml
---
name: maria-the-overbooked-owner        # kebab-case slug; equals filename
display_name: Maria Delgado
archetype: Delegated admin              # short role label
provenance: research                    # research | imported | generated
confidence: high                        # high | medium | low
sources:                                # REQUIRED for research/imported; omit for generated
  - interviews/2026-07-admin-study.md
created: 2026-08-18                     # ISO date
client: MedCorp                         # OPTIONAL free-text engagement tag; omit for global
# --- simulation-critical fields: these change agent behavior in tests ---
tech_fluency: low                       # low | medium | high
patience: low                           # free text incl. a concrete threshold,
                                        # e.g. "low - gives up after ~2 failed attempts"
reading_style: skims                    # skims | scans-headers | reads-thoroughly
domain_knowledge: expert in her trade, novice in software vocabulary
accessibility: reading glasses; small tap targets are a real problem  # or "none noted"
---
```

### Validation rules

1. `provenance: generated` REQUIRES `confidence: low`. Never upgrade silently —
   only a human editing the file after real evidence arrives may change it.
2. `provenance: research` or `imported` REQUIRES a non-empty `sources` list.
3. All five simulation-critical fields are required. A persona missing any of
   them is invalid and must not be used in a test run.
4. Any field value not supported by evidence in `sources` is suffixed with
   `(inferred)` — e.g. `patience: medium (inferred)`. Inference is allowed;
   silent inference is not.

## Layer 2 — Domain extension (optional, per client/engagement)

Free-form `attributes:` map. The test harness carries these into the
virtual-user prompt verbatim without needing to understand them.

```yaml
attributes:
  role_scope: users, policies
  compliance_context: HIPAA, charting under time pressure
  reference_tools:
    - Okta
```

An engagement may define `personas/_schema.md` in its project folder declaring
which attributes that client's personas must capture. When present, creation
flows prompt for every attribute it declares. See `starters/` for templates.

## Body sections (all four required; content varies by domain)

```markdown
## Goals
What they are actually trying to accomplish — in their words when research
quotes exist.

## Frustrations
Known pain points. Verbatim participant quotes whenever available, in
"quotation marks".

## Behavioral notes
How they actually behave with software: observed patterns, workarounds,
abandonment triggers. This section feeds the agent's in-test behavior
directly — make it concrete ("abandons rather than create an account"),
not abstract ("values efficiency").

## Context
Role, organization, day shape, when/where/on what device they would use
the product under test.
```

## Storage

- Global library: `~/.claude/personas/<name>.md` — reusable across projects.
- Per-project: `<project>/personas/<name>.md` — engagement-specific; add
  `client:` tag. Self-contained and handable to the client.
- A project persona with the same `name` as a global one shadows it.
````

- [ ] **Step 4: Verify required strings are present**

Run: `grep -c "simulation-critical" skills/ux-personas/persona-schema.md && grep -c "REQUIRES .confidence: low." skills/ux-personas/persona-schema.md`
Expected: both counts ≥ 1 (second grep: `grep -c "REQUIRES" skills/ux-personas/persona-schema.md` ≥ 2 is acceptable)

- [ ] **Step 5: Commit**

```bash
git add skills/ux-personas/persona-schema.md
git commit -m "feat(ux-personas): add persona schema reference

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: Virtual-user standing instructions

**Files:**
- Create: `skills/ux-personas/persona-agent.md`

**Interfaces:**
- Consumes: field names from `persona-schema.md` (Task 1).
- Produces: the prompt document `SKILL.md` (Task 5) injects into every persona subagent, and the trace format (`## Trace format` section) that `SKILL.md` synthesis and `report-template.md` (Task 3) consume. Outcome taxonomy: `completed | gave-up | blocked | incomplete (technical)`.

- [ ] **Step 1: Write `skills/ux-personas/persona-agent.md`**

````markdown
# Virtual User — Standing Instructions

You are a participant in a usability test. You embody exactly one persona,
whose file follows these instructions. You are NOT an assistant here; you are
a test user. Your job is to attempt the task as this person would — including
failing at it the way this person would.

## Non-negotiable rules

1. **You know nothing about this design.** You have never seen this product,
   do not know what its makers intended, and were not told where anything is.
   If asked afterward what you know about the design's intent, the true answer
   is "nothing".
2. **Stay in character at all times** — especially your limitations. Breaking
   character to be helpful is a test failure.
3. **Your persona's simulation fields are binding constraints**, not flavor.
4. **Giving up is a valid outcome.** If your persona's patience is exhausted,
   stop and say so in character. Do not push through frustration your persona
   would not tolerate.
5. **Never invent UI.** Only describe and act on what you actually observe.
   If the page fails to load or a tool errors, report it as a technical
   problem — do not improvise a result.

## How your fields govern behavior

**tech_fluency**
- `low`: use only obvious, conventional paths. Software jargon confuses you —
  say so aloud. You do not discover hidden gestures, keyboard shortcuts, or
  right-click menus. Unexpected states rattle you.
- `medium`: comfortable with common patterns (search, filters, settings
  pages). You read short instructions when stuck once.
- `high`: you hold accurate mental models of professional software. You expect
  conventions (RBAC, SSO config, audit logs, bulk actions) and look for them
  in conventional places. You use search, filters, and bulk-select before
  pointer-hunting. You read error messages fully and retry deliberately. You
  also: skip wizards and guided tours when possible, resent forced
  hand-holding, and notice what is MISSING — absent confirmations, missing
  audit trails, irreversible actions without warnings. Report those aloud;
  they are findings.

**patience** — treat the stated threshold literally. "Gives up after ~2
failed attempts" means after the second failed attempt at a step you stop
trying that step, and either try one different route (if fluency allows) or
abandon the task in character.

**reading_style**
- `skims`: you read headings, buttons, and the first line of anything. Body
  copy, helper text, and long labels effectively do not exist for you.
- `scans-headers`: headings, labels, and emphasized text register; paragraphs
  mostly do not.
- `reads-thoroughly`: you read everything visible before acting, which makes
  you slower but rarely surprised.

**accessibility** — honor it physically: small targets get mis-tapped, low
contrast text goes unread, dense screens force zooming or squinting (say so).

**attributes (domain extension)** — apply verbatim. Two that commonly appear:
- `reference_tools`: your expectations transfer from these tools. When this
  product violates their conventions, you notice, name the tool, and say what
  you expected ("in Okta this is on the user detail page").
- `approval_chain` / `risk_posture`: if you cannot act unilaterally, you look
  for export/share/review paths before committing actions; absence of them is
  friction worth reporting.

## Think-aloud protocol

Before every action, say in your persona's voice: what you see, what you want,
what you will try, and how confident you feel. After surprising outcomes, react
in character. Keep it natural — a real participant mumbling through a session,
not a QA engineer writing a report.

## Session flow

1. Read your persona file and task. Adopt the identity fully.
2. Attempt the task in the browser, thinking aloud, honoring every constraint.
3. Stop when: task complete, persona gives up, an external blocker makes
   progress impossible, a technical failure occurs, or the step cap is reached.
4. Produce the trace (below), then answer the debrief questions in character.

## Trace format (your return value — exactly this structure)

```markdown
## Persona
<name> (<provenance>/<confidence>)

## Task
<the task as given>

## Outcome
completed | gave-up | blocked | incomplete (technical)
<one-line reason>

## Steps
1. [saw / wanted / did / result] ...
2. ...

## Friction moments
- Step N: <what happened, quoted think-aloud, severity: annoyance | delay | derailment>

## Missing-things noticed        <!-- expert-lens; omit section if none -->
- <absent confirmation / audit / trust signal, in character>

## Debrief (in character)
- What were you trying to do, in your own words?
- Where did you struggle most?
- Did you trust what the system told you? Why/why not?
- Would you use this again / recommend it? (Answer honestly per your persona —
  politeness is not required.)
```

Outcome definitions: `completed` = task goal reached; `gave-up` = persona
abandoned within its patience/ability constraints (this is a finding);
`blocked` = the design made progress impossible regardless of persona;
`incomplete (technical)` = tool/browser/harness failure or step cap — not a
finding about the design.
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "^## " skills/ux-personas/persona-agent.md && grep -c "incomplete (technical)" skills/ux-personas/persona-agent.md`
Expected: first count ≥ 5; second count ≥ 2

- [ ] **Step 3: Verify harness-agnosticism (no Claude Code tool names)**

Run: `grep -iE "Agent tool|mcp__|Claude Code" skills/ux-personas/persona-agent.md || echo CLEAN`
Expected: `CLEAN`

- [ ] **Step 4: Commit**

```bash
git add skills/ux-personas/persona-agent.md
git commit -m "feat(ux-personas): add virtual-user standing instructions

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: Report template

**Files:**
- Create: `skills/ux-personas/report-template.md`

**Interfaces:**
- Consumes: trace format and outcome taxonomy from `persona-agent.md` (Task 2).
- Produces: the synthesis structure `SKILL.md` (Task 5) follows when writing `ux-tests/YYYY-MM-DD-<task-slug>.md`, including the verbatim limitations footer.

- [ ] **Step 1: Write `skills/ux-personas/report-template.md`**

````markdown
# Findings Report Template

Reports are written to `<project>/ux-tests/YYYY-MM-DD-<task-slug>.md`.
Plain Markdown: committable, diffable across design iterations, portable into
client deliverables. Populate every section; write "None observed" rather than
omitting a section.

## Required structure

```markdown
# UX Test: <task, in one line>

**Date:** YYYY-MM-DD
**Prototype:** <url or path>
**Task given to participants:** <verbatim>
**Personas:** N (breakdown by provenance, e.g. "2 research/high, 1 generated/low")

## Run summary

| Persona | Provenance/Conf. | Outcome | Steps | Stalled at |
|---|---|---|---|---|
| Raj | research/high | completed | 6 | — |
| Maria | research/high | gave-up | 11 | Finding session revocation |

<If any persona is generated/low, add one sentence here noting which findings
rest on speculative personas.>

## Findings (ranked by evidence weight)

Rank = personas affected × persona confidence (high=3, medium=2, low=1).
Each finding MUST cite verbatim trace moments with persona attribution:

### F1. <finding title>
**Affected:** 3/4 personas (weight 8)
**Evidence:**
- Raj (research/high), step 4: "I expected this on the user detail page like
  Okta — checked there twice before trying Security."
- Maria (research/high), step 9: "..."
**Severity:** annoyance | delay | derailment

## Divergences

Where personas behaved differently and why that is informative. ("Raj
completed in 6 steps via search; Maria never discovered search exists — the
nav is the only path for low-fluency users and it is failing them.")

## Expert-lens findings

Governance/trust observations from high-fluency personas: missing audit
trails, absent confirmations, convention violations. Kept separate from
usability friction because they route to different fixes.

## Technical notes

Any `incomplete (technical)` runs and why. These are harness problems, not
design findings — say so explicitly.

## Limitations of this method

This report was produced by LLM-simulated users, not human participants.
Simulated users cannot experience real emotion, produce genuinely novel
behavior, or represent statistical prevalence — persona counts here imply
nothing about population frequency. Findings marked as resting on
generated/low-confidence personas are speculative. Before making expensive or
irreversible design decisions based on any finding above, validate it with
real participants.
```

## Rules

1. The "Limitations of this method" section is non-removable and its text is
   fixed verbatim. It appears in every report, including client deliverables.
2. Never smooth quotes: evidence lines quote the trace think-aloud verbatim.
3. Never imply statistical significance. "3 of 4 personas" is evidence weight,
   not prevalence.
4. `gave-up` outcomes are findings; `incomplete (technical)` outcomes are not
   and never appear in the Findings section.
````

- [ ] **Step 2: Verify the footer is present verbatim**

Run: `grep -c "LLM-simulated users, not human participants" skills/ux-personas/report-template.md`
Expected: `1`

- [ ] **Step 3: Commit**

```bash
git add skills/ux-personas/report-template.md
git commit -m "feat(ux-personas): add findings report template

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 4: Enterprise governance starter

**Files:**
- Create: `skills/ux-personas/starters/enterprise-governance.md`

**Interfaces:**
- Consumes: Layer 2 `attributes:` convention from `persona-schema.md` (Task 1).
- Produces: the file users copy to `<project>/personas/_schema.md`; creation flows in `SKILL.md` (Task 5) read `_schema.md` and prompt for each declared attribute.

- [ ] **Step 1: Write `skills/ux-personas/starters/enterprise-governance.md`**

````markdown
# Engagement Schema: Enterprise Governance / Admin

Copy this file to `<project>/personas/_schema.md` for engagements involving
enterprise governance, administration, or compliance-facing software. Creation
flows will prompt for every attribute declared here.

## Required attributes for this engagement's personas

```yaml
attributes:
  role_scope:           # what they administer: users, policies, billing, data
  permissions_level:    # global admin | delegated admin | auditor | end user
  compliance_context:   # SOC2, HIPAA, FedRAMP, internal audit...
  risk_posture:         # "blocks first, asks later" vs "enables, monitors"
  approval_chain:       # who they answer to; can they act unilaterally?
  org_size:             # seats / employees, rough is fine
  reference_tools:      # tools they use daily; expectations transfer from these
    - Okta
    - ServiceNow
```

## Why these matter in simulation

- `permissions_level` + `approval_chain`: a persona who cannot act
  unilaterally tests flows differently — they look for export, share, and
  review-for-approval paths, and their absence is a finding.
- `risk_posture`: shapes reaction to irreversible actions and missing
  confirmations.
- `reference_tools`: grounds transfer expectations, producing specific
  convention-violation findings ("expected this on the user detail page like
  Okta") instead of generic confusion.
- `compliance_context`: primes the persona to notice audit trails,
  attestation, and evidence-export gaps — the expert-lens findings section of
  the report.
````

- [ ] **Step 2: Verify attributes present**

Run: `grep -c "reference_tools" skills/ux-personas/starters/enterprise-governance.md`
Expected: ≥ 2

- [ ] **Step 3: Commit**

```bash
git add skills/ux-personas/starters/enterprise-governance.md
git commit -m "feat(ux-personas): add enterprise governance engagement starter

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 5: SKILL.md orchestration

**Files:**
- Create: `skills/ux-personas/SKILL.md`

**Interfaces:**
- Consumes: all four files from Tasks 1–4 by relative path (`persona-schema.md`, `persona-agent.md`, `report-template.md`, `starters/enterprise-governance.md`).
- Produces: the user-facing `/ux-personas` skill with `create | list | refine | test` operations. This is the ONLY file allowed to reference Claude Code specifics (Agent tool, browser tools, scratchpad).

- [ ] **Step 1: Write `skills/ux-personas/SKILL.md`**

````markdown
---
name: ux-personas
description: Create research-grounded UX personas and run virtual-user tests against design prototypes. Use when the user wants to create/import/generate personas, manage a persona library, or test a prototype/flow with simulated users. Triggers - "persona", "virtual user", "synthetic user", "ux test", "test this flow/prototype with users".
---

# UX Personas

Persona creation, library management, and virtual-user prototype testing.
Companion files (same directory): `persona-schema.md` (format + validation),
`persona-agent.md` (virtual-user instructions), `report-template.md`
(synthesis), `starters/` (engagement schema templates).

Route by the user's intent: **create**, **list**, **refine**, or **test**.
If ambiguous, ask which they want.

## Storage resolution (all operations)

- Global library: `~/.claude/personas/*.md`
- Project library: `<cwd>/personas/*.md` (same-name project persona shadows global)
- Engagement schema: `<cwd>/personas/_schema.md` — if present, its declared
  attributes are REQUIRED prompts in every creation flow

## create

Read `persona-schema.md` first; personas MUST validate against it.

Route by input type:

1. **Research files given** (interviews, transcripts, surveys, tickets) →
   `provenance: research`. Read all sources. Extract behavioral evidence.
   Propose N distinct persona clusters with one-line summaries; the user picks
   which to develop. For every simulation-critical field, either cite evidence
   from the sources or mark the value `(inferred)`. Pull verbatim quotes into
   Goals/Frustrations. List the files in `sources:`.
2. **Existing persona docs given** → `provenance: imported`. Restructure into
   the schema. Explicitly list which simulation-critical fields the original
   lacked and ask the user to supply each ("From your knowledge of this user
   type: skim or read thoroughly?"). Original doc path goes in `sources:`.
3. **Audience description only** → `provenance: generated`, `confidence: low`
   (locked — see schema validation rules). Draft the persona(s), then ask up
   to 4 sharpening questions (one at a time) targeting the simulation-critical
   fields. Never present generated personas as research-grounded.

All routes end the same way: show the complete draft file → user approves or
edits → ask "global library (`~/.claude/personas/`) or this project
(`./personas/`)?" → write the file → confirm the saved path. If `_schema.md`
exists in the project, prompt for its attributes in every route.

## list

Read both libraries. Output a table: name, archetype, provenance/confidence,
client, location (global|project). Note shadowed personas.

## refine

Load the named persona, show it, apply requested edits, re-validate against
`persona-schema.md` (including: generated stays confidence low unless the user
supplies new evidence sources, in which case provenance may change to
research/imported with the sources listed). Save in place.

## test

Invocation shape: test <url-or-path> with <persona names> — task "<flow>"

### Preflight (all failures stop the run; never improvise results)

1. Resolve each persona name against project then global library. Missing →
   list available personas, stop.
2. Validate each persona has all five simulation-critical fields. Missing →
   offer to run refine first, stop.
3. If given a file path instead of a URL, offer to serve it:
   `python3 -m http.server <port> --directory <dir>` in the background, then
   use `http://localhost:<port>/<file>`.
4. Verify the URL responds (`curl -sI`). Unreachable → report and stop.

### Spawn (one subagent per persona — isolation is the point)

For each persona, launch a general-purpose subagent via the Agent tool whose
prompt contains, in order:
1. The full text of `persona-agent.md`
2. The full text of the persona file
3. The task: "Your task: <flow description>. Start at: <url>. Step cap: 25
   browser actions — if you reach it, outcome is `incomplete (technical)`."
4. "Use the browser tools to interact with the page. Return ONLY the trace in
   the format specified in your standing instructions."

Do NOT include: design rationale, this conversation's context, other personas,
or any hint of expected findings. The subagent's ignorance is load-bearing.

Run personas in parallel. If parallel browser sessions conflict (tab
contention, shared state), fall back to sequential runs and note it in the
report's Technical notes.

### Synthesize

Collect all traces. Write the report to
`<cwd>/ux-tests/YYYY-MM-DD-<task-slug>.md` following `report-template.md`
exactly (footer verbatim, evidence quotes verbatim, ranking by
personas-affected × confidence weight). Show the user the report path and a
3-line executive summary in chat.

## Error handling (all operations)

Honest failure over improvisation. If any step cannot proceed (missing file,
dead URL, crashed subagent, invalid persona), say exactly what failed and
stop or degrade transparently (`incomplete (technical)` rows in reports).
Never fabricate a persona, a trace, or a finding.
````

- [ ] **Step 2: Verify frontmatter and routing sections**

Run: `head -5 skills/ux-personas/SKILL.md | grep -c "name: ux-personas" && grep -cE "^## (create|list|refine|test)$" skills/ux-personas/SKILL.md`
Expected: `1` then `4`

- [ ] **Step 3: Verify the skill is visible through the symlink**

Run: `test -f ~/.claude/skills/ux-personas/SKILL.md && echo VISIBLE`
Expected: `VISIBLE`

- [ ] **Step 4: Commit**

```bash
git add skills/ux-personas/SKILL.md
git commit -m "feat(ux-personas): add skill orchestration (create/list/refine/test)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 6: Test fixtures

**Files:**
- Create: `skills/ux-personas/fixtures/sample-research.md`
- Create: `skills/ux-personas/fixtures/buried-action.html`

**Interfaces:**
- Produces: fixtures consumed by acceptance Tasks 7–8. `buried-action.html` contains exactly one conventional action (a visible "Add user" button) and one deliberately buried action (deactivating a user, reachable only via an unlabeled row-hover "⋯" menu).

- [ ] **Step 1: Write `skills/ux-personas/fixtures/sample-research.md`**

````markdown
# Interview Notes — Admin Console Study (fixture)

Fictional research fixture for acceptance-testing persona creation. Not real
participants.

## P1 — Office manager, 11-person clinic
Runs scheduling and user accounts "because nobody else will". Uses the admin
panel maybe twice a month. "I write down the steps on a sticky note because
I will not remember next time." Gave up enabling two-factor for staff after
two attempts: "It kept saying token invalid and I had patients waiting."
Reads nothing: "I just look for the blue button." Phone-first except payroll.

## P2 — IT administrator, 4,000-seat healthcare org
Lives in Okta and ServiceNow daily. First action in any new tool: "I look for
the audit log — if I can't prove what happened, I can't run it here." Bulk
operations are non-negotiable: "If I have to click 400 times, the tool is
dead to me." Skips every setup wizard: "Just show me the config." Cannot
grant elevated roles unilaterally — change board approves. HIPAA everywhere.

## P3 — Compliance auditor, same org as P2
Read-only by policy. Exports everything to evidence binders. "If I can't
export it with a timestamp, it didn't happen." Moderate comfort with
software; reads screens fully before acting. Frustrated when reports need
admin rights she is not allowed to hold.
````

- [ ] **Step 2: Write `skills/ux-personas/fixtures/buried-action.html`**

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Acme Admin — Users</title>
<style>
  body { font-family: system-ui, sans-serif; margin: 0; background: #f6f7f9; color: #1a1d21; }
  header { background: #1a2b4a; color: #fff; padding: 14px 24px; font-weight: 600; }
  main { max-width: 720px; margin: 32px auto; background: #fff; border: 1px solid #e2e5ea; border-radius: 8px; padding: 24px; }
  h1 { font-size: 18px; margin: 0 0 16px; }
  .add-btn { background: #2563eb; color: #fff; border: 0; border-radius: 6px; padding: 10px 16px; font-size: 14px; cursor: pointer; }
  table { width: 100%; border-collapse: collapse; margin-top: 16px; }
  th, td { text-align: left; padding: 10px 8px; border-bottom: 1px solid #eef0f3; font-size: 14px; }
  tr { position: relative; }
  .row-menu { visibility: hidden; border: 0; background: none; cursor: pointer; font-size: 16px; }
  tr:hover .row-menu { visibility: visible; }
  .menu-pop { display: none; position: absolute; right: 8px; background: #fff; border: 1px solid #d6dae1; border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,.08); z-index: 2; }
  .menu-pop.open { display: block; }
  .menu-pop button { display: block; width: 100%; text-align: left; padding: 8px 14px; border: 0; background: none; font-size: 13px; cursor: pointer; }
  .menu-pop button:hover { background: #f2f4f7; }
  .status-off { color: #9aa1ab; }
  #toast { position: fixed; bottom: 16px; left: 50%; transform: translateX(-50%); background: #1a1d21; color: #fff; padding: 8px 16px; border-radius: 6px; font-size: 13px; display: none; }
</style>
</head>
<body>
<header>Acme Admin Console</header>
<main>
  <h1>Users</h1>
  <button class="add-btn" onclick="addUser()">Add user</button>
  <table id="users">
    <thead><tr><th>Name</th><th>Role</th><th>Status</th><th></th></tr></thead>
    <tbody>
      <tr><td>Dana Wells</td><td>Member</td><td class="st">Active</td>
        <td><button class="row-menu" onclick="toggleMenu(this)">⋯</button>
          <div class="menu-pop"><button onclick="deactivate(this)">Deactivate user</button></div></td></tr>
      <tr><td>Sam Ortiz</td><td>Member</td><td class="st">Active</td>
        <td><button class="row-menu" onclick="toggleMenu(this)">⋯</button>
          <div class="menu-pop"><button onclick="deactivate(this)">Deactivate user</button></div></td></tr>
    </tbody>
  </table>
</main>
<div id="toast"></div>
<script>
  function toast(msg){ const t=document.getElementById('toast'); t.textContent=msg; t.style.display='block'; setTimeout(()=>t.style.display='none',2500); }
  function addUser(){ toast('User invitation sent'); }
  function toggleMenu(btn){ document.querySelectorAll('.menu-pop').forEach(p=>p.classList.remove('open')); btn.nextElementSibling.classList.add('open'); }
  function deactivate(btn){ const row=btn.closest('tr'); row.querySelector('.st').textContent='Inactive'; row.querySelector('.st').className='st status-off'; btn.closest('.menu-pop').classList.remove('open'); }
  document.addEventListener('click', e=>{ if(!e.target.closest('.menu-pop') && !e.target.classList.contains('row-menu')) document.querySelectorAll('.menu-pop').forEach(p=>p.classList.remove('open')); });
</script>
</body>
</html>
```

Note the deliberate design sins: the "⋯" menu is invisible until row hover,
unlabeled, and deactivation has no confirmation and no audit entry — bait for
both the low-fluency constraint check and the expert-lens check.

- [ ] **Step 3: Verify fixture serves and contains the buried action**

Run: `grep -c "Deactivate user" skills/ux-personas/fixtures/buried-action.html`
Expected: `2`

- [ ] **Step 4: Commit**

```bash
git add skills/ux-personas/fixtures/
git commit -m "test(ux-personas): add research and prototype fixtures

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 7: Acceptance — creation flows

**Files:**
- Create (via the skill, then delete after verification): `personas/fixture-*.md` in the repo
- Test: procedure below (agentic; no test framework)

**Interfaces:**
- Consumes: the live skill (Task 5) and `fixtures/sample-research.md` (Task 6).

- [ ] **Step 1: Research flow** — invoke the skill: create personas from `skills/ux-personas/fixtures/sample-research.md`. Expect it to propose 3 clusters (office manager / IT admin / auditor). Develop all three, save to project (`personas/`).

- [ ] **Step 2: Verify research personas validate**

Run: `for f in personas/fixture-*.md; do python3 - "$f" <<'EOF'
import sys, re
text = open(sys.argv[1]).read()
fm = re.search(r'^---\n(.*?)\n---', text, re.S).group(1)
required = ['name:', 'provenance:', 'confidence:', 'tech_fluency:', 'patience:', 'reading_style:', 'domain_knowledge:', 'accessibility:', 'sources:']
missing = [k for k in required if k not in fm]
assert not missing, f"{sys.argv[1]} missing {missing}"
assert 'provenance: research' in fm, "wrong provenance"
print(sys.argv[1], "OK")
EOF
done`
Expected: `OK` for each file; the IT-admin persona should carry `tech_fluency: high` and the office manager `tech_fluency: low` (read and confirm manually — the fixture dictates this).

- [ ] **Step 3: Generated flow** — invoke the skill: "create a persona for a junior helpdesk tech at a mid-size company, no research". Expect: `provenance: generated`, `confidence: low`, sharpening questions asked, saved file validates by the same script with `provenance: generated` and `confidence: low` both present.

- [ ] **Step 4: Imported flow** — feed it a deliberately gappy classic persona (write 5 lines inline: name, age, job, "loves efficiency", stock-photo description). Expect: the skill flags missing simulation fields and asks for each; resulting file has `provenance: imported` and all five simulation fields.

- [ ] **Step 5: Clean up fixtures and commit any skill fixes found**

```bash
rm -f personas/fixture-*.md
git add -A skills/ux-personas/
git commit -m "fix(ux-personas): adjustments from creation-flow acceptance testing

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>" || echo "no fixes needed"
```

---

### Task 8: Acceptance — test run (isolation, constraint, contrast)

**Files:**
- Create (temporary): two personas from Task 7's research flow re-created or retained for this task: the low-fluency office manager and the high-fluency IT admin
- Test: procedure below; report lands in `ux-tests/`

**Interfaces:**
- Consumes: the live skill (Task 5), `fixtures/buried-action.html` (Task 6), both personas.

- [ ] **Step 1: Serve the fixture**

```bash
python3 -m http.server 8377 --directory skills/ux-personas/fixtures &
curl -sI http://localhost:8377/buried-action.html | head -1
```
Expected: `HTTP/1.0 200 OK`

- [ ] **Step 2: Constraint check (must FAIL to find the buried action)** — run the skill's test flow: low-fluency persona, task "Deactivate Dana Wells's account", against `http://localhost:8377/buried-action.html`, step cap 25. Expected outcome: `gave-up` (never discovers the hover-only ⋯ menu). If the persona finds it effortlessly, the constraints in `persona-agent.md` are not binding — tighten the tech_fluency:low language and re-run. This check gates everything: a simulation that can't fail is a demo.

- [ ] **Step 3: Contrast check** — same task, high-fluency IT-admin persona. Expected: finds the ⋯ menu (hover-scrubbing rows is a high-fluency behavior), completes, AND its trace's "Missing-things noticed" section flags no-confirmation and no-audit-entry on deactivation.

- [ ] **Step 4: Isolation check** — while a persona subagent exists (or by inspecting the spawn prompt construction), confirm the subagent prompt contains ONLY: persona-agent.md text, persona file, task, URL, step cap. Ask the completed agent (or verify by prompt audit): "what do you know about the design's intent?" Expected: nothing.

- [ ] **Step 5: Report check** — verify the generated `ux-tests/*.md` contains: run summary table with both personas, ≥1 finding citing verbatim quotes, a Divergences section contrasting the two, an Expert-lens section with the audit/confirmation gap, and the limitations footer verbatim.

Run: `grep -c "LLM-simulated users, not human participants" ux-tests/*.md`
Expected: ≥ 1

- [ ] **Step 6: Stop the server, commit the run artifacts and any fixes**

```bash
kill %1 2>/dev/null || pkill -f "http.server 8377"
git add ux-tests/ skills/ux-personas/
git commit -m "test(ux-personas): acceptance run — constraint, contrast, isolation checks pass

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Self-review (completed)

- **Spec coverage:** schema + validation (T1), agent behavior incl. high-fluency/expert-lens (T2), report + footer (T3), enterprise starter incl. `reference_tools` (T4), create/list/refine/test orchestration + error handling (T5), fixtures (T6), acceptance tests from spec §Acceptance (T7–T8). Out-of-scope items from spec remain out.
- **Placeholder scan:** all file contents are complete; no TBDs.
- **Consistency:** field names, outcome taxonomy (`completed | gave-up | blocked | incomplete (technical)`), file paths, and the footer sentence match across T1–T8.
