# security-kit

A Claude Code skill that installs a curated bundle of third-party security review and threat-modeling skills into a project on demand. No hooks by default. Fully reversible. XDG-compliant.

## What it does

Security review with Claude Code works best when the model has the right skills loaded for the task. `security-kit` does three things:

1. **Installs curated skill bundles** into `.claude/skills/` of the current project, with variants for different needs (minimal, sec-review, threat-modeling, full).
2. **Ships battle-tested prompts** for the common workflows (broad review, PR-diff triage, threat modeling, shell-string audit).
3. **Orchestrates runs.** `/sec-kit scan` runs the installed skills in the correct sequence. `/sec-kit smart-scan` fingerprints the project first and curates a prompt tailored to its shape.

Raptor (the offensive-security harness) is opt-in via `/sec-kit install-raptor` — it ships a SessionStart hook and overrides project-level `CLAUDE.md`, so it's kept separate from the default flow.

## Quick start

In any project directory:

```
/sec-kit install            # install the full variant (no hooks)
/sec-kit scan               # run the installed skills against this project
/sec-kit uninstall          # clean up when done
```

For a project where you want Claude to pick the right skills itself:

```
/sec-kit smart-scan         # analyzes project shape, writes a tailored scan prompt
```

## Commands

| Command | Purpose |
|---|---|
| `/sec-kit list` | List every skill the kit offers + what's currently installed in this project |
| `/sec-kit install [variant]` | Install skills. Variants: `minimal`, `sec-review`, `threat-modeling`, `full` (default). No hooks. |
| `/sec-kit install-hooks <name...>` | Opt-in hook installer from a whitelist: `gh-cli`, `modern-python` |
| `/sec-kit install-raptor` | Opt-in raptor framework. Installs SessionStart hook, `/raptor-*` commands, CLAUDE.md override. |
| `/sec-kit uninstall [variant]` | Remove installed skills. Default removes all sec-kit-managed skills. |
| `/sec-kit uninstall-hooks <name...>` | Remove specified hooks |
| `/sec-kit uninstall-raptor` | Remove raptor framework from the project |
| `/sec-kit scan` | Run all installed skills in the correct sequence against this project |
| `/sec-kit smart-scan` | Fingerprint the project, then curate a scan prompt tailored to the project's shape |

## Install variants

| Variant | Skills | Use when |
|---|---|---|
| `minimal` | 5 skills (the T6 chain): audit-context-building, variant-analysis, supply-chain-risk-auditor, insecure-defaults, fp-check | Quick adversarial review of a small/medium codebase |
| `sec-review` | 13 skills: minimal + differential-review, semgrep-rule-creator, constant-time-analysis, zeroize-audit, codeql, semgrep, sarif-parsing, owasp-security | Standard security review workflow |
| `threat-modeling` | 9 skills: tm-init, tm-threats, tm-drift, tm-full, tm-report, tm-verify, tm-compliance, tm-status, tm-tests | STRIDE/PASTA threat modeling, PR-drift detection |
| `full` (default) | All 22: sec-review ∪ threat-modeling | General-purpose installation |

Deliberately excluded from all variants:
- `entry-point-analyzer` — smart-contract-only (Solidity/Vyper/Solana/Move/TON/CosmWasm), not general-purpose
- `firebase-apk-scanner` — active internet scanner
- fr33d3m0n's threat-modeling skill — ships a PostToolUse Write hook (use raptor's opt-in path if you want something similar)
- Trail of Bits plugins with hooks (`gh-cli`, `modern-python`, `skill-improver`, `firebase-apk-scanner`, `second-opinion`) — opt-in via `install-hooks`

## Where things live (XDG-compliant)

Skill sources are cloned to `$XDG_DATA_HOME/security-kit/sources/` (defaults to `~/.local/share/security-kit/sources/` on Linux/macOS). Symlinks from `.claude/skills/` point into that directory, so a single `git pull` updates every project using those skills.

Resolution order for source paths:
1. `--sources-dir` CLI flag (per-invocation)
2. `.security-kit.yaml` in the project root → `sources_dir:`
3. `~/.config/security-kit/config.yaml` → `sources_dir:`
4. `$SECURITY_KIT_HOME` environment variable
5. `$XDG_DATA_HOME/security-kit/sources/` (default)
6. `~/.local/share/security-kit/sources/` (XDG fallback)

## Prompts

Prewritten, battle-tested prompts live in `prompts/` of this repo and are installed alongside the skills at `.security-kit/prompts/` in your project:

- `full-review.md` — the T6 multi-skill chain (audit-context → hunt → variants → supply-chain → insecure-defaults → fp-check)
- `pr-diff-triage.md` — differential-review → tm-drift chain for "does this PR need a threat model update"
- `threat-model.md` — full STRIDE workflow via tm-*
- `shell-string-audit.md` — the F4-gap closer, hunts template-literal shell command builders that every adversarial skill misses

## Trust model

- The kit itself is pure markdown + `bash` commands. No Python, no compiled code.
- Skills installed from third-party sources (Trail of Bits, OWASP, josemlopez, fr33d3m0n) are symlinked unmodified; you can `cat` any `.claude/skills/*/SKILL.md` to see exactly what Claude will read.
- Hooks are opt-in. Nothing runs automatically after `install` without your explicit `install-hooks` or `install-raptor`.
- Raptor's opt-in path prints what it will install (SessionStart hook, CLAUDE.md override, slash commands) and waits for confirmation.

## Uninstall is complete

`/sec-kit uninstall` removes every symlink the kit created. `uninstall-hooks` and `uninstall-raptor` each clean their own additions. The project-local `.security-kit/` directory is removed. Skill sources in `~/.local/share/security-kit/` stay put (they're shared across projects); `/sec-kit purge` removes those too if you want a full wipe.

## License

Apache-2.0.
