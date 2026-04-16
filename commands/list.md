# /sec-kit list

Show every skill the kit offers plus what is currently installed in this project.

## What to do

1. Read all variant manifests from the kit:
   - `manifests/minimal.yaml`
   - `manifests/sec-review.yaml`
   - `manifests/threat-modeling.yaml`
   - `manifests/full.yaml`
   - `manifests/hooks.yaml`

2. Check what is currently installed in this project:
   - `.claude/skills/` — list existing entries (all symlinks presumed managed by the kit; non-symlinks are user's own skills)
   - `.security-kit/installed.yaml` — if present, it is the authoritative record of kit-managed installations

3. Present a table to the user:

```
Available variants
  minimal         5 skills — quick adversarial review
  sec-review     13 skills — standard security review
  threat-modeling 9 skills — STRIDE/PASTA threat modeling
  full           22 skills — full kit (default)

Currently installed in this project
  variant: full (installed 2026-04-16)
  skills: audit-context-building, variant-analysis, ... (all 22)
  hooks:  (none)
  raptor: not installed

Available opt-in hooks
  gh-cli          PATH shim + PreToolUse redirect for github.com URLs → gh
  modern-python   PATH shim suggesting uv over pip

Opt-in raptor framework
  raptor          SessionStart hook, CLAUDE.md override, /raptor-* commands
                  (separate from the skill variants; use `install-raptor`)
```

4. If nothing is installed, say so and suggest `install` or `smart-scan`.

## Skill descriptions (short)

Load from `docs/installed-skills.md` for one-line descriptions of each skill.
