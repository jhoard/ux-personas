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
