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

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Orchestration (the only Claude Code-specific file) |
| `persona-schema.md` | Persona format + validation rules (portable) |
| `persona-agent.md` | Virtual-user standing instructions (portable) |
| `report-template.md` | Findings report structure (portable) |
| `starters/enterprise-governance.md` | Per-engagement schema starter for enterprise governance/admin work |
| `fixtures/` | Acceptance-test fixtures (sample research + a deliberately flawed prototype) |
| `docs/` | Design spec and implementation plan |
| `examples/` | A real findings report from the acceptance run |

Everything except `SKILL.md` is harness-agnostic by design — porting to another agent harness means rewriting only the orchestration page.

## Lineage & limitations

Inspired by [UXAgent](https://github.com/neuhai/UXAgent) (CHI 2025). Simulated users cannot replace research with real participants — every report this skill produces says so, non-removably. They are strongest at early flow/findability testing and weakest at concept validation.
