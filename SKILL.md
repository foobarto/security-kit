---
name: security-kit
description: |
  Install, manage, and orchestrate a curated bundle of third-party Claude Code security review
  and threat-modeling skills in a project. Use this skill when the user says any of: "set up
  security tools", "install security skills", "audit this project", "run security scan",
  "threat model this project", "/sec-kit", "sec-kit install", "sec-kit scan", "sec-kit smart-scan".
  Provides install/uninstall of skill bundles (minimal, sec-review, threat-modeling, full),
  opt-in hook management, opt-in raptor framework integration, and two scan orchestrators:
  a standard scan that runs installed skills in sequence, and a smart-scan that fingerprints
  the project first and curates a tailored prompt.
---

# security-kit

This skill manages security tooling for the current project. It does not perform the security
review itself — it installs the skills that do, ships the prompts that drive them, and
orchestrates their execution.

## When to use

- "install security tools in this project" → `/sec-kit install`
- "run a security scan" / "audit this codebase" → `/sec-kit scan` (if installed) or `/sec-kit smart-scan`
- "what security skills are installed" → `/sec-kit list`
- "install raptor" / "install offensive security framework" → `/sec-kit install-raptor`
- "install pentagi" / "install pentesting platform" → `/sec-kit install-pentagi`
- "remove the security skills" / "clean up" → `/sec-kit uninstall`

## Command reference

Every subcommand is documented in a dedicated file under `commands/`:

| Subcommand | File | Brief |
|---|---|---|
| `list` | `commands/list.md` | Show all skills the kit offers + what's installed locally |
| `install` | `commands/install.md` | Install a skill variant (minimal/sec-review/threat-modeling/full) |
| `install-hooks` | `commands/install-hooks.md` | Opt-in hook installation from a whitelist |
| `install-raptor` | `commands/install-raptor.md` | Opt-in raptor framework install (confirms before acting) |
| `uninstall` | `commands/uninstall.md` | Remove installed skills |
| `uninstall-hooks` | `commands/uninstall-hooks.md` | Remove installed hooks |
| `uninstall-raptor` | `commands/uninstall-raptor.md` | Remove raptor framework |
| `install-pentagi` | `commands/install-pentagi.md` | Opt-in PentAGI pentesting platform install |
| `uninstall-pentagi` | `commands/uninstall-pentagi.md` | Remove PentAGI wrapper and deployment files |
| `scan` | `commands/scan.md` | Run installed skills in the correct order against the project |
| `smart-scan` | `commands/smart-scan.md` | Fingerprint the project, then curate a tailored scan prompt |

When the user invokes a subcommand, read the relevant command file and follow its instructions exactly.

## Resolution: where skill sources live

The kit expects skill source repos at `$XDG_DATA_HOME/security-kit/sources/` (defaults to
`~/.local/share/security-kit/sources/`). If they're not there, `install` will clone them.
Resolution order for the sources directory:

1. `--sources-dir <path>` flag in the invocation
2. `.security-kit.yaml` → `sources_dir:` in the project root
3. `~/.config/security-kit/config.yaml` → `sources_dir:`
4. `$SECURITY_KIT_HOME` env var
5. `$XDG_DATA_HOME/security-kit/sources/` (respects XDG)
6. `~/.local/share/security-kit/sources/` (final fallback)

On first run in a new environment, `install` creates the directory and clones the repos it needs:

| Repo | Sources dir |
|---|---|
| trailofbits-skills | `<SOURCES>/trailofbits-skills/` |
| claude-code-owasp | `<SOURCES>/claude-code-owasp/` |
| threat-modeling-toolkit | `<SOURCES>/threat-modeling-toolkit/` |
| anthropic-cybersecurity-skills | `<SOURCES>/anthropic-cybersecurity-skills/` (cybersecurity variant) |
| vxcontrol/pentagi | `<SOURCES>/pentagi/` (only on `install-pentagi`) |
| gadievron/raptor | `<SOURCES>/raptor/` (only on `install-raptor`) |

## Project-local footprint

After an install, the project has:

- `.claude/skills/<skill-name>` — symlinks to skill sources (one per installed skill)
- `.security-kit/` — kit-managed state and prompts:
  - `.security-kit/installed.yaml` — lockfile listing what was installed and when
  - `.security-kit/prompts/*.md` — copied from `prompts/` in this skill
  - `.security-kit/last-scan.md` — last scan prompt (if `smart-scan` was used)

Uninstall removes all of the above.

## Prompt templates

Pre-built prompts in `prompts/` of this skill:

- `full-review.md` — T6 multi-skill chain (ground-truth tested, produced the best run in the comparative audit)
- `pr-diff-triage.md` — differential-review + tm-drift chain
- `threat-model.md` — STRIDE workflow via tm-*
- `shell-string-audit.md` — closes the F4 cross-stack miss (every adversarial skill misses template-literal shell-command builders)

## Philosophy

- **Hooks require explicit opt-in.** Nothing runs automatically after `install`.
- **Everything is reversible.** Every command has an uninstall counterpart.
- **Pure markdown.** No Python, no compiled code. You can read every file the kit uses.
- **Skills come from upstream unmodified.** The kit symlinks into third-party skill repos; run `git -C <sources-dir>/<repo> pull` to update.
- **XDG-compliant.** Follows the XDG Base Directory spec so it plays well with containers, sandboxes, and users who relocate `$XDG_DATA_HOME`.
