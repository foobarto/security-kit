# Prompt guide — which prompt for which situation

The kit ships four prompt templates in `prompts/`. After `install`, they live at `.security-kit/prompts/` in your project.

| Situation | Use |
|---|---|
| "Should I install this third-party extension / library?" | `full-review.md` |
| "Audit this codebase before we ship it" | `full-review.md` |
| "Review this PR" | `pr-diff-triage.md` |
| "Build a threat model for this project" | `threat-model.md` |
| "Find shell-injection issues" | `shell-string-audit.md` (or included as a step in `full-review.md`) |

## Picking a prompt

### Full review (`full-review.md`)

The multi-skill chain that produced the best run in the comparative audit. Appropriate for:

- One-shot security assessment of an unknown codebase
- Pre-release audit of your own project
- Decision-grade review (install / don't install, ship / don't ship)

Cost: ~50-100k tokens on a 15k LOC project, roughly linear with codebase size.

### PR-diff triage (`pr-diff-triage.md`)

Chains `differential-review` → `tm-drift`. Use when:

- You have an existing threat model (`.threatmodel/` present)
- You're reviewing an incoming PR or a commit range
- You want a benign/needs-TM-update/needs-new-threats verdict

If no threat model exists, this prompt falls back to differential-review alone and recommends creating a baseline.

Cost: ~10-30k tokens depending on commit count.

### Threat modeling (`threat-model.md`)

Full STRIDE workflow via `tm-*`. Produces:

- Machine-readable `.threatmodel/state/*.json`
- Mermaid architecture + trust-boundary diagrams
- Executive summary + full risk report

Use when:

- You need a formal threat model for compliance or audit
- You want a durable artifact team can maintain across versions
- You'll use `tm-drift` later for PR triage

Cost: ~30-50k tokens for initial build; `tm-drift` re-runs are cheap.

### Shell-string audit (`shell-string-audit.md`)

Targeted hygiene pass for template-literal subprocess builders. Always include as a step in any larger scan — every adversarial skill misses this pattern unless explicitly told to hunt it.

Cost: trivial. 1-2k tokens.

## Prompt variables

All prompts use `{{VARIABLE}}` placeholders. `smart-scan` fills these automatically from the fingerprint; manual use requires substitution. See individual prompt files for the variable list each uses.

## Customization

Project-local prompts at `.security-kit/prompts/` can be edited freely — they're your copy. To reset to upstream defaults, run `/security-kit install` again (overwrites local copies).

To share custom prompts across projects, edit the originals in `~/Dokumenty/security-kit/prompts/` (or wherever the kit is checked out) and re-run `install` in each project.
