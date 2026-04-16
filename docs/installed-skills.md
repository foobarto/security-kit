# Installed skills — one-line reference

Every skill the kit can install, with a short description and primary use case.

## Sec-review skills (Trail of Bits + OWASP)

| Skill | Purpose |
|---|---|
| `audit-context-building` | Line-by-line architectural context before hunting. Run first in any review. |
| `variant-analysis` | After finding a bug, check whether the same pattern recurs elsewhere. |
| `supply-chain-risk-auditor` | Audit dependencies + install scripts + runtime remote fetchers for supply-chain attack vectors. |
| `insecure-defaults` | Hunt hardcoded secrets, weak auth, fail-open configs, permissive defaults. |
| `fp-check` | Validate suspected bugs to eliminate false positives. Use as a post-pass with explicit evidence requirements. |
| `differential-review` | Security-focused review of PR diffs / commit ranges. Blast-radius + trust-boundary + new-surface classification. |
| `semgrep-rule-creator` | Write custom Semgrep rules when you need to hunt a specific pattern across large codebases. |
| `constant-time-analysis` | Timing-side-channel audits for cryptographic code. |
| `zeroize-audit` | Memory zeroization audits in Rust (for code handling secrets). |
| `codeql` | Run CodeQL interprocedural taint tracking. Requires `codeql` binary installed. |
| `semgrep` | Run Semgrep with parallel subagents. Requires `semgrep` binary installed. |
| `sarif-parsing` | Parse SARIF output from semgrep/codeql. Support skill. |
| `owasp-security` | OWASP Top 10 (2025-2026) + ASVS 5.0 lens. Good for web-facing code. |

## Threat-modeling skills (josemlopez)

| Skill | Purpose |
|---|---|
| `tm-init` | Bootstrap a new threat model from code. Creates `.threatmodel/` state. |
| `tm-threats` | Enumerate STRIDE threats per trust boundary. |
| `tm-drift` | Detect drift against a prior threat-model snapshot. Answer "does this change invalidate the TM?" |
| `tm-full` | Run the complete threat-modeling workflow (init → threats → verify → compliance → report). |
| `tm-report` | Generate the final markdown threat model report. |
| `tm-verify` | Cross-check threats against mitigations. |
| `tm-compliance` | Map threats to compliance frameworks (SOC2, ISO-27001, etc.). |
| `tm-status` | Show current state of the threat model in this project. |
| `tm-tests` | Generate test cases from threat model. |

## Opt-in hooks (via `install-hooks`)

| Hook | Purpose | Risk |
|---|---|---|
| `gh-cli` | Redirects github.com URLs to the `gh` CLI for better structured output | Low |
| `modern-python` | Suggests `uv` when `pip` is used | Low |

## Opt-in raptor framework (via `install-raptor`)

Raptor is not a skill but a full offensive-security workspace harness. Installing it adds:

- ~17 agents (crash analysis, OSS forensics, exploitability validation)
- ~20 slash commands (`/raptor-scan`, `/raptor-fuzz`, `/raptor-web`, `/agentic`, `/oss-forensics`, etc.)
- A SessionStart hook that prints a banner
- ~400 lines appended to project `CLAUDE.md`
- A Python CLI pipeline (`raptor.py agentic --repo <path>`) that orchestrates Semgrep + CodeQL + Claude

Use when you want runnable PoCs and unified-diff patches for disclosure.

## Deliberately excluded

These are available in the source repos but not installed by any variant:

| Skill / plugin | Reason |
|---|---|
| `entry-point-analyzer` | Smart-contract-specific (Solidity/Vyper/Solana/Move/TON/CosmWasm). Doesn't fit general-purpose codebases. |
| `firebase-apk-scanner` | Active internet scanner. Too invasive for a default install. |
| `skill-improver` | Aggressive self-continuing Stop-hook loop. Burns tokens. |
| `second-opinion` | Auto-launches Codex MCP if present. Third-party dependency. |
| fr33d3m0n threat-modeling | Ships a PostToolUse Write hook. Use raptor if you want a hook-bearing framework. |

If you want any of these, clone the upstream repo yourself and symlink manually. The kit intentionally doesn't automate their installation.
