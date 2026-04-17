# Sources — where skills come from

The kit maintains a local copy of every skill repo it uses. Skills in your project `.claude/skills/` are symlinks into these sources.

## Default location

`$XDG_DATA_HOME/security-kit/sources/` — defaults to `~/.local/share/security-kit/sources/` on Linux/macOS.

This follows the XDG Base Directory Specification. The same path is used by `uv`, `pipx`, `mise`, and other modern tools.

## Resolution order

The kit resolves the sources dir in this order (first match wins):

1. `--sources-dir <path>` flag (per-invocation)
2. `.security-kit.yaml` in the project root → `sources_dir:` field
3. `~/.config/security-kit/config.yaml` → `sources_dir:` field
4. `$SECURITY_KIT_HOME` environment variable
5. `$XDG_DATA_HOME/security-kit/sources/` (default)
6. `~/.local/share/security-kit/sources/` (XDG fallback if `$XDG_DATA_HOME` unset)

## Auto-detect existing clones

If `~/tools/sec-review/_sources/*` or `~/tools/threat-modeling/_sources/*` already exist (from manual installs prior to the kit), the kit will detect and reuse them instead of re-cloning. Auto-detect paths in priority order:

1. `~/.local/share/security-kit/sources/` (XDG default)
2. `~/tools/sec-review/_sources/` + `~/tools/threat-modeling/_sources/` (legacy layout)
3. Any path configured via the resolution order above

This lets users migrate without losing their existing clones.

## Repos cloned

| Repo | URL | Used by |
|---|---|---|
| `trailofbits-skills` | https://github.com/trailofbits/skills.git | sec-review variant, hooks |
| `claude-code-owasp` | https://github.com/agamm/claude-code-owasp.git | sec-review variant |
| `threat-modeling-toolkit` | https://github.com/josemlopez/threat-modeling-toolkit.git | threat-modeling variant |
| `anthropic-cybersecurity-skills` | https://github.com/mukul975/Anthropic-Cybersecurity-Skills.git | cybersecurity variant (53 curated skills from 754 total) |
| `pentagi` | https://github.com/vxcontrol/pentagi.git | install-pentagi only (opt-in pentesting platform) |
| `raptor` | https://github.com/gadievron/raptor.git | install-raptor only (opt-in) |

Clones are always `--depth 1`. Update via:

```bash
cd ~/.local/share/security-kit/sources/trailofbits-skills
git pull --depth 1
```

All symlinked projects pick up the update immediately.

## Config file format

`~/.config/security-kit/config.yaml`:

```yaml
version: 1
sources_dir: /data/shared/security-kit-sources
# Alternative names for repos — only use if you've forked them
repo_urls:
  trailofbits-skills: https://github.com/your-fork/skills.git
```

Project-level override: `.security-kit.yaml` at the project root. Same format. Useful when a project needs a specific fork or pinned version.

## Offline / air-gapped environments

Clone the source repos by hand on a connected machine, rsync them to the air-gapped machine at `$XDG_DATA_HOME/security-kit/sources/`, then use the kit normally — it'll detect the existing clones and skip the git clone step.

## Purge

`/sec-kit purge` removes the sources directory. Warning: affects every project that has sec-kit skills installed, since their symlinks will break. Re-run `install` in each project after a purge to restore.
