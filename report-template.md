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
**Persona-agent model:** <model id that played the personas>
**Synthesis model:** <model id that wrote this report>
**Last fidelity check:** <YYYY-MM-DD, traits verified — or "not run">

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

## Method provenance

<One line naming the persona-agent model. If it shares a model family with the
synthesis model, add: "The same model family played the personas and wrote this
synthesis — findings are subject to backbone bias, where a model grades its own
performance." Omit the second sentence only when the families genuinely differ.>

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
5. Carry `(invented)` display-name markers from persona files into the report's
   run summary table, so invented names are never mistaken for real participants.
6. Model provenance is recorded from what was actually dispatched, never from
   memory or assumption. Persona behavior moves with the model, so a report
   without it cannot be reproduced or compared against a later run.
7. State the backbone-bias line whenever the persona-agent and synthesis models
   share a family. It is a real limit on the finding, not boilerplate — the
   model that played the user is grading its own performance.
