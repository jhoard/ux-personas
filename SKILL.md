---
name: ux-personas
description: Create research-grounded UX personas and run virtual-user tests against design prototypes. Use when the user wants to create/import/generate personas, manage a persona library, or test a prototype/flow with simulated users. Triggers - "persona", "virtual user", "synthetic user", "ux test", "test this flow/prototype with users".
---

# UX Personas

Persona creation, library management, and virtual-user prototype testing.
Companion files (same directory): `persona-schema.md` (format + validation),
`persona-agent.md` (virtual-user instructions), `report-template.md`
(synthesis), `fidelity-check.md` (two-arm behavior validation),
`starters/` (engagement schema templates).

Route by the user's intent: **create**, **list**, **refine**, **test**, or
**verify**. If ambiguous, ask which they want.

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
   Goals/Frustrations. List the files in `sources:`. If sources are anonymized
   (P1/P2...), invent a natural display_name but mark it `display_name: <Name> (invented)`
   so no one mistakes it for a real participant.
2. **Existing persona docs given** → `provenance: imported`. Restructure into
   the schema. Explicitly list which simulation-critical fields the original
   lacked and ask the user to supply each ("From your knowledge of this user
   type: skim or read thoroughly?"). Original doc path goes in `sources:`.
   If the input was pasted text rather than a file, record it as
   `pasted: <short description> (<date>)` in sources:. Default imported personas
   to `confidence: medium` — grounded in someone's real documentation but not
   verifiable from sources you can read. The user may raise or lower it.
3. **Audience description only** → `provenance: generated`, `confidence: low`
   (locked — see schema validation rules). Draft the persona(s), then ask
   sharpening questions (one at a time) for any simulation-critical field you
   could not plausibly draft from the description — draft first, ask only where
   genuinely uncertain, and stop when all five fields hold defensible values.
   Never present generated personas as research-grounded.

All routes end the same way: for EACH persona, show the complete draft file and
get approval or edits before moving to the next. Then ask ONCE for the batch:
"global library (`~/.claude/personas/`) or this project (`./personas/`)?" —
allowing per-persona overrides if the user wants a split. Write the files and
confirm each saved path. If `_schema.md` exists in the project, prompt for its
attributes in every route.

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

Record the model you dispatched each persona subagent with. The report states
it as fact, not from memory — a persona-agent model is part of the evaluation
configuration, and results move with it.

### Synthesize

Collect all traces. Write the report to
`<cwd>/ux-tests/YYYY-MM-DD-<task-slug>.md` following `report-template.md`
exactly (footer verbatim, evidence quotes verbatim, ranking by
personas-affected × confidence weight), including the persona-agent and
synthesis model fields. If those two models share a family, add the
backbone-bias line the template specifies — you played the user and are now
grading the performance. Show the user the report path and a 3-line executive
summary in chat.

## verify

Proves persona simulation fields actually change behavior — and that they have
not been tightened so far the persona can no longer succeed. Read
`fidelity-check.md` first; it defines pole pairs, the judge rubric, and what
each failure means.

Invocation shape: verify [<trait> | all]. Traits with shipped pole pairs:
`tech_fluency`, `reading_style`, `patience`. If asked for a trait without a
pair (`domain_knowledge`, `accessibility`), say plainly that no arena can
decide it yet rather than inventing one.

For each pole pair:

1. **Synthesize the two variants** into a scratch directory — NEVER the user's
   persona library. Identical in every field except the trait under test, which
   takes each pole. Both need all five simulation-critical fields to be valid
   per `persona-schema.md`.
2. **Serve the arena** from `fixtures/` and verify it responds, same preflight
   discipline as `test`.
3. **Run the arms one at a time**, each in a fresh browser tab, using the exact
   `test` spawn prompt — the same `persona-agent.md` text, the same
   persona-file-only isolation, the same step cap. The harness must exercise the
   real path or it proves nothing about it. Do NOT parallelize arms: they share
   browser state, and a contaminated arm (input typed by the other arm, page
   swapped mid-task) is `incomplete (technical)`, never a pass. See
   `fidelity-check.md`.
4. **Judge each arm** with a subagent on a DIFFERENT model from the persona
   agent. Give it the trace and the declared pole only — never the arena's
   source, never which arm is expected to complete the task. Ask for
   `expressed` / `correctly-suppressed` / `violated` plus the trace line that
   decided it. Remember `gave-up` is the correct outcome for a constrained arm.

Report a table: trait, pole, outcome, verdict, evidence quote. Pass requires
BOTH arms correct. On failure, name the direction and the rule: constrained arm
succeeded → that `persona-agent.md` rule is decoration, tighten it; capable arm
failed → it is over-tight, loosen it. Never report a pass for an arm that did
not actually run — a fixture that will not serve or a crashed arm is
`incomplete (technical)`.

## Error handling (all operations)

Honest failure over improvisation. If any step cannot proceed (missing file,
dead URL, crashed subagent, invalid persona), say exactly what failed and
stop or degrade transparently (`incomplete (technical)` rows in reports).
Never fabricate a persona, a trace, or a finding.
