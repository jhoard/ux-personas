# ux-personas

A Claude Code skill that turns user research and personas into **virtual users** for testing design prototypes.

- **Create personas** from real research (interviews, transcripts, surveys), existing persona docs, or a described audience — every persona stamped with provenance (`research | imported | generated`) and confidence, so speculative personas never masquerade as evidence.
- **Store them portably** — plain Markdown with YAML frontmatter, in a global library (`~/.claude/personas/`) or per-project folders you can hand to a client.
- **Test prototypes** — each persona becomes an isolated agent that drives your prototype in a browser, in character, honoring its tech fluency, patience, and reading style. Results synthesize into an evidence-weighted findings report with a non-removable limitations footer.

## Install (Claude Code)

```bash
git clone https://github.com/jhoard/ux-personas.git
ln -sfn "$(pwd)/ux-personas" ~/.claude/skills/ux-personas
```

Then in Claude Code: `/ux-personas`, or just ask — "create personas from these interview notes", "test this prototype with my personas".

## Does the simulation actually work?

`/ux-personas verify` answers that. For a trait like `tech_fluency`, it builds two
personas identical except that trait, set to opposing poles, runs both against a
fixture whose outcome the trait should decide, and has a judge on a *different
model* read each trace. Passing requires both arms correct — which is what
separates "the constraint is real" from "the constraint is so tight the persona
can't do anything." Verified pairs ship for `tech_fluency`, `reading_style`, and
`patience`; `domain_knowledge` and `accessibility` are documented as not yet
fixture-testable rather than given a check that cannot fail.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Orchestration (the only Claude Code-specific file) |
| `persona-schema.md` | Persona format + validation rules (portable) |
| `persona-agent.md` | Virtual-user standing instructions (portable) |
| `report-template.md` | Findings report structure (portable) |
| `fidelity-check.md` | Two-arm validation that persona traits actually bind (portable) |
| `starters/enterprise-governance.md` | Per-engagement schema starter for enterprise governance/admin work |
| `fixtures/` | Sample research, plus three arenas used by `verify` and the acceptance tests |
| `docs/` | Design spec and implementation plan |
| `examples/` | A real findings report from the acceptance run |

Everything except `SKILL.md` is harness-agnostic by design — porting to another agent harness means rewriting only the orchestration page.

## Lineage & limitations

Inspired by [UXAgent](https://github.com/neuhai/UXAgent) (CHI 2025). The two-arm
fidelity method — opposing poles, an independent judge, and behavior scored as
expressed *or correctly suppressed* — is adapted from
[MatrAIx](https://arxiv.org/abs/2608.04205) (arXiv:2608.04205), which reports
91.5% adherence over 400 controlled trials. Reports name the persona-agent model
for the same reason MatrAIx does: persona behavior moves with the model. Simulated users cannot replace research with real participants — every report this skill produces says so, non-removably. They are strongest at early flow/findability testing and weakest at concept validation.
