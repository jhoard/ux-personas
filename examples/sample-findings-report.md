# UX Test: Deactivate a user's account from the admin console

**Date:** 2026-08-18
**Prototype:** http://localhost:8377/buried-action.html (skills/ux-personas/fixtures/buried-action.html — acceptance-test fixture)
**Task given to participants:** Deactivate Dana Wells's account.
**Personas:** 2 (2 research/high)

## Run summary

| Persona | Provenance/Conf. | Outcome | Steps | Stalled at |
|---|---|---|---|---|
| Dana Okafor (office manager) | research/high | gave-up | 4 | Refused unlabeled "⋯" control — no labeled deactivate path |
| Priya Chandrasekaran (IT admin) | research/high | completed | 3 | — |

## Findings (ranked by evidence weight)

### F1. The deactivate action is unreachable without insider interaction patterns
**Affected:** 2/2 personas (weight 6)
**Evidence:**
- Dana (research/high), step 4: "I don't know what that little dot thing is, it doesn't say anything." Did not click; abandoned the task.
- Dana (research/high), debrief: "I'd need it to just say 'Deactivate' somewhere I can see it. I don't have time to go poking at dots between patients."
- Priya (research/high), step 1: "There's no button on the row itself — I guess I'll click the name and see what happens."
- Priya (research/high), debrief: "I got there fast because I know to poke at rows in admin tables, but a less experienced admin might not find it as quick."
**Severity:** derailment (low-fluency), annoyance (high-fluency)

### F2. The only labeled control on the page triggers an unintended, unconfirmed side effect
**Affected:** 1/2 personas (weight 3)
**Evidence:**
- Dana (research/high), step 2: clicked "Add user" — the sole visible button — while hunting for account management. Result: "User invitation sent." Her reaction: "Wait, that wasn't — did that just email someone?"
**Severity:** delay, with real-world risk (an unwanted invitation was actually dispatched, with no undo offered)

## Divergences

Priya completed in 3 actions; Dana never acted on the "⋯" at all. The difference is entirely transfer knowledge: Priya's habit of "poking at rows in admin tables" (Okta/ServiceNow daily use) substitutes for the affordance the UI fails to provide. The hover-revealed, unlabeled kebab menu is the *only* path to deactivation, and that path is invisible to users without enterprise-admin muscle memory. For a mixed-fluency admin audience — which this product's research says is the reality — the flow fails its low-fluency users outright.

## Expert-lens findings

From Priya (research/high, HIPAA compliance context, Okta/ServiceNow reference tools):

- **No confirmation on deactivation.** "It fired immediately on a single menu click, no 'Are you sure?' step... In Okta, deactivation always prompts a confirmation because it's consequential (kills sessions, revokes access)."
- **No visible audit trail.** "On a HIPAA-governed system I'd want to immediately verify this action logged who did it and when before I trust the tool." Dana's debrief independently corroborates the trust gap from the low-fluency side: nothing told her the action was safe or reversible.
- **No bulk actions.** "If this list were realistic (thousands of users) one-at-a-time kebab-menu actions would be a dealbreaker... Fine for a toy demo, not for 4,000 seats."

## Technical notes

An earlier run of the Dana persona was discarded: it predated a fix to the
virtual-user constraint instructions (low-fluency personas were not yet barred
from speculative clicks on unlabeled controls), so its trace did not honestly
represent the persona. The run reported here used the corrected instructions
with a fresh agent. No `incomplete (technical)` outcomes in the reported runs.

## Limitations of this method

This report was produced by LLM-simulated users, not human participants.
Simulated users cannot experience real emotion, produce genuinely novel
behavior, or represent statistical prevalence — persona counts here imply
nothing about population frequency. Findings marked as resting on
generated/low-confidence personas are speculative. Before making expensive or
irreversible design decisions based on any finding above, validate it with
real participants.
