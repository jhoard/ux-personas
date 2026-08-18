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
