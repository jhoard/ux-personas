# UX Personas — Design Spec

**Date:** 2026-08-18
**Status:** Approved design, pending implementation plan
**Owner:** James Hoard

## What this is

A Claude Code skill (`ux-personas`) that turns user research and personas into **virtual users** that test design prototypes and report findings. Three capabilities:

1. **Create personas** from real research, existing persona docs, or a described audience
2. **Store personas** as portable Markdown files in a global library and/or per-project folders
3. **Test prototypes** by spawning one isolated persona-conditioned subagent per persona, each driving the prototype in a browser and thinking aloud in character; results are synthesized into an evidence-weighted findings report

Inspired by [UXAgent](https://github.com/neuhai/UXAgent) (CHI 2025) but built native to Claude Code: no separate Python app, no separate API billing, personas in the user's own format.

## Design principles

- **Portability first.** All domain knowledge (persona schema, virtual-user instructions, report templates, the personas themselves) lives in plain Markdown files any agent harness can consume. Only `SKILL.md` orchestration is Claude Code-specific; porting to Codex or another harness means rewriting ~one page of glue.
- **Provenance is a first-class guardrail.** Every persona is stamped `research | imported | generated` with a confidence level, and every report carries those labels forward. Speculative personas never masquerade as research. This is the defense against the documented synthetic-user validity trap (sycophancy, false confidence).
- **Simulation fields over documentation fluff.** Persona fields exist because they change agent behavior in a test, not for decoration.
- **Honest failure over improvisation.** Missing inputs, dead URLs, and crashed subagents surface as labeled errors, never as fabricated results.

## Persona schema

One persona = one Markdown file. Two layers.

### Layer 1 — Universal core (domain-agnostic, never changes)

```yaml
---
name: maria-the-overbooked-owner        # slug, filename
display_name: Maria Delgado
archetype: Delegated admin              # short role label
provenance: research                    # research | imported | generated
confidence: high                        # high | medium | low
sources:                                # required for research/imported
  - interviews/2026-07-admin-study.md
created: 2026-08-18
client: MedCorp                         # optional free-text engagement tag
# --- simulation-critical fields (drive agent behavior) ---
tech_fluency: low                       # low | medium | high
patience: low                           # e.g. gives up after ~2 failed attempts
reading_style: skims                    # skims | scans-headers | reads-thoroughly
domain_knowledge: expert in her trade, novice in software vocabulary
accessibility: reading glasses; small tap targets are a real problem
---
```

Body sections (universal, content differs by domain):

- **Goals** — what they're trying to accomplish, in their words where research exists
- **Frustrations** — known pain points, verbatim quotes when available
- **Behavioral notes** — observed software behavior ("abandons rather than create an account")
- **Context** — role, day shape, when/where they use the product

Fields the research didn't support are marked `inferred` inline rather than silently guessed.

### Layer 2 — Domain extension (per client/engagement, open-ended)

Free-form `attributes:` map carried into the agent prompt verbatim:

```yaml
attributes:
  role_scope: users, policies
  compliance_context: HIPAA
```

An engagement may define `personas/_schema.md` declaring which attributes that client's personas must capture; `/persona create` then prompts for them.

### Enterprise governance starter (`starters/enterprise-governance.md`)

Default extension template for enterprise governance/admin design work:

```yaml
attributes:
  role_scope:           # what they administer: users, policies, billing, data
  permissions_level:    # global admin | delegated admin | auditor | end user
  compliance_context:   # SOC2, HIPAA, FedRAMP, internal audit...
  risk_posture:         # "blocks first, asks later" vs "enables, monitors"
  approval_chain:       # who they answer to; can they act unilaterally?
  org_size:
  reference_tools:      # tools they use daily; expectations transfer from these
    - Okta
    - ServiceNow
```

`risk_posture` and `approval_chain` shape test behavior: a persona who can't act unilaterally seeks export/share-for-approval paths. `reference_tools` grounds transfer expectations ("expected this on the user detail page like Okta").

## Persona creation flows

Single entry point `/persona create`, routing by input:

| Input | Provenance | Flow |
|---|---|---|
| Research files (interviews, transcripts, surveys, tickets) | `research` | Extract behavioral evidence → propose N distinct persona clusters → user picks → every simulation field cites evidence or is marked `inferred` → verbatim quotes preserved |
| Existing persona docs (any format) | `imported` | Restructure into schema → flag gaps (classic personas usually lack tech_fluency/patience/reading_style) → ask the user to fill them |
| Audience description (a sentence or two) | `generated`, confidence locked to `low` | Draft personas → brief sharpening interview → never silently upgraded |

All flows end identically: show draft → user approves/edits → ask "global library or this project?" → save → confirm path. If `_schema.md` exists, its attributes are prompted in every flow.

Other commands: `/persona list` (browse both libraries), `/persona refine <name>` (guided edit).

## Test runs

`/persona test <url> --personas <names...> --task "<flow description>"`

1. **Preflight** — validate URL reachable (offer to serve a local directory if not), validate each persona file has simulation fields (prompt to fill if not)
2. **Spawn** — one subagent per persona via the Agent tool. Each subagent receives ONLY: the persona file, the task, `persona-agent.md` (standing virtual-user instructions), and browser tools. It does not see the design rationale or the main conversation — isolation is what keeps testing honest.
3. **Act in character** — the agent drives the prototype (click, scroll, type, read), thinking aloud in the persona's voice, honoring constraints:
   - `tech_fluency: low` → conventional paths only, confusion at jargon, no hidden-gesture discovery
   - `tech_fluency: high` + `reference_tools` → expects RBAC/SSO/audit/bulk conventions, uses search and bulk actions, reads errors fully, retries; but skips wizards, resents hand-holding, and flags missing power/trust features ("revoked access but no session-termination confirmation — where's the audit entry?")
   - `patience: low` → genuinely gives up after its stated threshold; giving up is a valid, valuable outcome
   - `reading_style: skims` → misses body copy; buried CTAs go unfound
4. **Return trace** — structured: steps taken, think-aloud transcript, outcome (`completed | gave-up | blocked | incomplete (technical)`), friction moments, post-task in-character debrief
5. **Parallelism** — multiple personas run as parallel subagents; falls back to sequential if browser contention corrupts traces

Step cap per agent prevents wandering; hitting it yields `incomplete (technical)`, distinct from `gave-up` (which is a finding).

## Synthesis & reporting

Output: `ux-tests/YYYY-MM-DD-<task-slug>.md` in the project. Plain Markdown — committable, diffable across design iterations, portable into client deliverables.

1. **Run summary table** — per persona: outcome, steps, stall point, provenance/confidence. "Blocked (research/high)" reads differently from "blocked (generated/low)" and the report says so.
2. **Findings ranked by evidence weight** — friction clustered across personas, ordered by (personas affected × persona confidence), each citing verbatim trace moments.
3. **Divergences** — where personas behaved differently and why that's informative ("Raj completed in 6 steps via search; Maria never discovered search exists").
4. **Expert-lens findings** — governance/trust observations (missing audit trails, absent confirmations, convention violations) reported separately from usability friction; they route to different fixes.
5. **Standing limitations footer** — auto-included, non-removable: synthetic testing cannot capture real emotion, novel behavior, or statistical significance; findings driving expensive decisions warrant real-participant validation.

## File layout

```
~/.claude/skills/ux-personas/
  SKILL.md                      # orchestration — the only harness-specific file
  persona-schema.md             # universal core + extension rules
  persona-agent.md              # virtual-user standing instructions
  report-template.md            # synthesis structure + limitations footer
  starters/
    enterprise-governance.md    # _schema.md starter to copy per engagement

~/.claude/personas/             # global library
<project>/personas/             # engagement personas + optional _schema.md
<project>/ux-tests/             # findings reports
```

## Error handling

| Failure | Behavior |
|---|---|
| URL unreachable | Offer to serve the directory locally; never "test" from imagination |
| Persona missing simulation fields | Prompt to fill before running; no silent guessing |
| Subagent crash / step cap | Row reads `incomplete (technical)`; partial runs still report, clearly labeled |
| Parallel browser contention | Fall back to sequential runs |

## Acceptance tests (for the skill itself)

1. **Creation flows** — one persona per provenance path; all schema-compliant
2. **Isolation check** — ask a persona subagent what it knows about the design's intent; correct answer is nothing
3. **Constraint check** — a `patience: low, tech_fluency: low` persona vs. a prototype with one deliberately buried action must *fail* to find it; effortless success means constraints aren't binding
4. **Contrast check** — low-fluency and high-fluency personas on the same task must produce visibly different traces and findings

The constraint check is the gate between a simulation and a demo.

## Out of scope (v1)

- Static wireframe/screenshot testing (browser-driven prototypes only; wireframe critique can come later)
- Thousand-persona batch runs (UXAgent remains the tool for that scale)
- Quantitative statistics (n is small; the report language avoids implying significance)
- Multi-user/collaborative scenario simulation
