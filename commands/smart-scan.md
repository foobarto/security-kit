# /sec-kit smart-scan

Fingerprint the current project, then curate a scan prompt tailored to its shape. Use when you don't know which skill variant fits, or you want to skip irrelevant skills.

## What to do

This is a two-phase command. Phase 1 analyzes the project; Phase 2 writes and executes a curated prompt.

---

## Phase 1 — Project fingerprint

Collect signals about the project without making security claims yet. Use Read, Glob, Grep, Bash.

### 1.1 Language & framework detection

- `package.json` present → Node.js ecosystem
  - `engines.vscode` or `contributes.commands` → **VS Code extension**
  - `next`, `react`, `vue`, `svelte` in deps → **web frontend**
  - `express`, `fastify`, `hapi`, `koa` → **Node backend**
- `requirements.txt` / `pyproject.toml` / `setup.py` → Python
  - `django`, `flask`, `fastapi` → web backend
  - `click`, `typer` → CLI
- `Cargo.toml` → Rust; look for `actix-web`, `axum`, `rocket` (web), `clap` (CLI), `tokio` (async)
- `go.mod` → Go; look for `net/http`, `gin`, `echo`
- `foundry.toml` / `hardhat.config.*` / `truffle-config.js` / `anchor` → **smart contracts**
- `pubspec.yaml` → Flutter/Dart
- `Gemfile` → Ruby
- `build.gradle` / `pom.xml` → JVM

### 1.2 Attack surface signals

Run parallel greps for sink-shaped code patterns. Note file:line counts for each:

| Signal | Grep pattern | Matters for |
|---|---|---|
| Webview/iframe | `webview`, `iframe`, `postMessage`, `Uri\.parse`, `openExternal` | Webview message handling |
| Remote fetches | `fetch\(`, `axios`, `got\(`, `https?.request`, `curl` | Supply-chain, data exfil |
| Subprocess | `execSync`, `spawn\(`, `child_process`, subprocess module calls | Shell injection (F4-shape) |
| IPC | `WebSocket`, `createServer`, `listen\(`, `net\.connect`, `socket\.io` | Auth boundary review |
| Crypto | `crypto\.`, `randomBytes`, `Math\.random`, `sign\(`, `verify\(` | Crypto misuse, constant-time |
| Auth flows | `oauth`, `OAuth`, `refresh_token`, `access_token`, `SecretStorage`, `keychain` | Credential handling |
| File I/O | `readFileSync`, `writeFile`, `fs\.`, `path\.join`, `os\.homedir` | Path traversal, cross-boundary reads |
| Deserialization | `JSON\.parse`, `yaml\.load`, object-deserialization libs, `Marshal` | Injection via parsed data |

### 1.3 Dependency shape

- Count of direct runtime deps
- Count of transitive deps (via lockfile — `npm ls`, `pip show`, etc.)
- Any packages with <1y old publication and <100 downloads → flag for supply-chain attention
- Any `postinstall`/`preinstall`/`prepare` scripts in `package.json` or transitively → flag

### 1.4 Target type inference

Combine the above into one of:

- `vscode-extension` — has `engines.vscode`
- `web-backend` — web framework + routes
- `web-frontend` — SPA framework, no backend
- `cli-tool` — CLI framework, no network server
- `library` — no entrypoint, published as a module
- `service` — has Dockerfile, systemd, or similar
- `smart-contract` — Foundry/Hardhat/Anchor/etc.
- `mixed` — multiple of the above

### 1.5 Write fingerprint to `.security-kit/fingerprint.md`

Format:

```markdown
# Project fingerprint — <project-name>

**Date:** <ISO>
**Type:** vscode-extension
**Languages:** TypeScript (primary), minor JS
**LOC (src):** ~15,000

## Frameworks
- vscode-extension-api (deps: `@types/vscode`)
- sql.js
- cron-parser

## Attack surface signals
| Signal | Count | Notes |
|---|---|---|
| Webview/postMessage | 47 | Primary UI |
| Remote fetches | 3 | announcement JSON, Google quota API, OAuth |
| Subprocess | 11 | hunter.ts, WSL helpers |
| IPC (WebSocket) | 1 | cockpit-tools integration |
| OAuth | yes | Google installed-app flow |
| Crypto | minor | Math.random for nonces |

## Dependencies
- Direct runtime: 2 (cron-parser, sql.js)
- Transitive: ~40
- Postinstall scripts: 1 (scripts/install-hooks.sh, dev-only)

## Inferred target type: vscode-extension
```

---

## Phase 2 — Curate the scan prompt

Based on the fingerprint, pick which skills to invoke and what to emphasize. Use this decision matrix:

### Skill selection

| Fingerprint signal | Skills to include | Skills to skip |
|---|---|---|
| Always | audit-context-building, variant-analysis, insecure-defaults, fp-check | — |
| Has dependencies | supply-chain-risk-auditor | — |
| Has remote fetches / webview | owasp-security | — |
| Has crypto with custom primitives | constant-time-analysis, zeroize-audit | — |
| Is smart-contract | — | everything above; tell user to use dedicated Solidity/etc. skills instead (kit doesn't bundle them) |
| Multiple commits / git history | differential-review (offer on recent commits) | — |
| Has architecture docs | (extra context for audit-context-building) | — |

### Prompt seeding

Inject fingerprint-specific hunt targets:

- **Webview detected** → "Trace every postMessage handler from the webview to extension-host sinks. Enumerate commandId/URL flows from remote fetch → webview → host."
- **Subprocess detected** → "Audit every subprocess invocation (template-literal shell commands with `${...}` interpolation) for injection fragility. Flag even if inputs are static today — a future change routing tainted data into the builder becomes an immediate sink."
- **OAuth detected** → "Audit OAuth flow for PKCE usage, state randomness (cryptographic vs Math.random), callback origin checks, hardcoded client secrets."
- **WebSocket IPC** → "Check WS authentication (handshake tokens, HMAC on messages), peer PID verification, server.json / config-file-controlled endpoints."
- **File I/O outside workspace** → "Review every path-building operation; check for symlink-follow, traversal, TOCTOU."
- **JSON/YAML from remote** → "Check for prototype pollution, ReDoS in validation, type-confusion between parsed types."

### Write the curated prompt

Write to `.security-kit/scan-prompt-<timestamp>.md`:

```markdown
# Curated scan prompt — <project-name>

Generated by /sec-kit smart-scan.
Target: <absolute path>
Type: <inferred>
LOC: <count>

## Skills to invoke (in order)
1. audit-context-building — produce architectural map
2. <other skills per decision matrix>
6. fp-check — validate findings

## Focus areas (based on fingerprint)
- <seeded hunt target 1>
- <seeded hunt target 2>
- ...

## Standard closers (always included)
- F4 shell-string audit: <link to shell-string-audit.md>
- Triage output: file:path:line + severity + attack scenario (1-2 sentences) + fix (1-2 sentences)

## Full prompt
<final prompt, ready to paste into Claude>
```

### Offer to execute

Ask the user:

```
Curated prompt saved to .security-kit/scan-prompt-<timestamp>.md.

Options:
  1. Execute now (I'll run the prompt myself)
  2. Show me the prompt, I'll review and run it later
  3. Execute with raptor (requires /sec-kit install-raptor)

Which?
```

On option 1, proceed to execute the prompt using the Skill tool for each invoked skill.

## Failure modes

- **No skills installed yet** → smart-scan still works (fingerprint is useful in its own right), but ask the user to run `/sec-kit install` or offer to install the variant best-matched to the fingerprint (e.g., `sec-review` for most projects, `threat-modeling` only if the user explicitly asks for a threat model).
- **Fingerprint is ambiguous** → surface the ambiguity to the user; let them pick the target type manually before writing the prompt.
- **Huge codebase** (>100k LOC) → offer to restrict to specific subdirectories before proceeding; full scans can be token-expensive.
