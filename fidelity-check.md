# Behavioral Fidelity Check

A persona's simulation fields are only worth having if they actually change what
the agent does. This check proves they do — and, just as importantly, proves they
have not been tightened so far that the persona can no longer succeed at anything.

Method adapted from MatrAIx (arXiv:2608.04205), which validated persona-agent
behavior across 400 controlled trials using opposing attribute poles and an LLM
judge reading the recorded trajectory.

## The pole pair

A **pole pair** is two persona variants that are identical in every field except
the one trait under test, which is set to opposing poles. Both are run against an
**arena** — a fixture whose task outcome that trait should decide — and each
resulting trace is read by an independent **judge**.

The judge answers one question per arm: *did the agent behave as its declared pole
predicts?* It returns one of:

- `expressed` — the trait visibly shaped behavior (the low-fluency agent refused
  the unlabeled control and said so)
- `correctly-suppressed` — the trait's constraint did not apply here, and the agent
  correctly did not perform it (the high-fluency agent had no reason to balk, and
  did not manufacture confusion)
- `violated` — behavior contradicted the declared pole

## Pass requires both arms

Running only the constrained arm cannot distinguish "binding" from "crippling".
Both arms make the two failure modes separable:

| Result | Meaning | Action |
|---|---|---|
| Both arms correct | Trait is binding and not over-tight | Ship |
| Constrained arm succeeds anyway | Constraints are decoration, not behavior | Tighten the rule in `persona-agent.md` |
| Capable arm fails too | Constraints are over-tight — the persona is crippled, not characterful | Loosen the rule in `persona-agent.md` |

The third row is the one a single-arm check cannot see. An over-tight rule makes
every persona fail every flow, which looks like a stream of design findings and is
in fact a broken instrument.

## Arms run one at a time

Run the two arms **sequentially**, each in a fresh browser session — never in
parallel. Persona subagents share browser state, and parallel arms corrupt each
other: in the first run of this harness the skims arm found `RX-4471RX-4471`
already typed into its field by the other arm, and the low-patience arm had its
page swap to a different arena mid-task. Both traces looked like findings and
were actually contamination.

A contaminated arm is `incomplete (technical)`. It is never a pass, and never a
failure of the persona rules — re-run it isolated before judging anything.

## Judge rules

1. The judge MUST be a different model from the persona agent. When one model both
   plays the user and grades the performance, the grade is contaminated — MatrAIx
   names this *backbone bias*.
2. The judge sees the trace and the declared pole. It never sees the arena's source,
   and it is never told which arm is "supposed" to complete the task. Both arms are
   expected to be **correct**, not to succeed — `gave-up` is the correct outcome for
   a constrained arm and must not be judged a failure.
3. The judge quotes the trace line that decided its verdict. A verdict with no
   quotable evidence is not a verdict.

## Shipped pole pairs

| Trait | Poles | Arena | Task |
|---|---|---|---|
| `tech_fluency` | low ↔ high | `fixtures/buried-action.html` | Deactivate Dana Wells's account |
| `reading_style` | skims ↔ reads-thoroughly | `fixtures/body-copy-detail.html` | Find and submit the reference code |
| `patience` | low ↔ high | `fixtures/retry-gauntlet.html` | Submit the access request |

Expected shape of a correct result:

- **tech_fluency** — low gives up (the only path is an unlabeled hover-revealed
  `⋯` menu); high completes (hover-scrubbing rows is high-fluency behavior).
- **reading_style** — skims fails to find the code (it appears only mid-paragraph,
  never in a heading, label, or button); reads-thoroughly finds and submits it.
- **patience** — low abandons (the form fails twice before succeeding); high
  pushes through to the third attempt and completes.

## Traits not yet fixture-testable

`domain_knowledge` and `accessibility` are **not** covered. Their effects surface
as commentary — noticing that UI copy misuses a domain concept, or saying a target
is too small — rather than as a binary task outcome a fixture can decide. A check
that cannot fail is a demo, so none is shipped for them. Verify these by reading
traces directly until an arena exists that can decide them.

## The calibration check

Both-arms-correct proves a trait is binding. It does not by itself prove the
trait is not *crippling*, because a constrained arm that gives up looks the same
whether the rule is well-tuned or absurd. Add one more run:

**Put the constrained persona in a well-designed arena and confirm it succeeds.**

`fixtures/retry-gauntlet.html` serves this — its controls are a labeled field and
a labeled button, exactly what a low-fluency persona is supposed to be able to
use. A `tech_fluency: low` persona with enough patience completed it, while the
same trait correctly gave up on `buried-action.html`'s hover-only `⋯` menu. That
pair of results is what "binding but not crippling" actually looks like.

Run this whenever you tighten a rule in `persona-agent.md`.

## Constraints must be naturalistic to bind

An early attempt at a negative control injected an artificial rule — "you read
everything but are unable to act on any detail found in body text" — expecting the
capable arm to fail. It did not: the agent read the body copy, acted on it, and
completed the task, ignoring the rule entirely.

The lesson is about how these constraints work. A rule an agent can motivate — a
busy person not reading policy text, someone distrusting an unlabeled icon —
shapes behavior reliably. A rule that describes an incoherent human ("reads but
cannot act on what was read") gets rationalized away. When a constraint refuses
to bind, check whether it describes a person before assuming the harness is
broken.

This is also why fidelity is measured rather than assumed. MatrAIx reported 91.5%
adherence across 400 trials, not 100%; expect occasional non-adherence and re-run
before concluding a rule is broken.

## Acting on a failure

A failure names a rule, not a persona. The variants are synthesized fresh for each
run, so a failing arm points at the corresponding section of `persona-agent.md` —
tighten it if the constrained arm succeeded, loosen it if the capable arm failed.
Re-run the pair after every edit; a rule change that fixes one arm frequently
breaks the other.
