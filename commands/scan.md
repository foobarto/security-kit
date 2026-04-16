# /sec-kit scan

Run the installed security skills against the current project, in the correct sequence.

## What to do

### Step 1 — Check what is installed

Read `.security-kit/installed.yaml`. If absent or empty, tell the user to run `/sec-kit install` or `/sec-kit smart-scan` first and stop.

### Step 2 — Pick the right prompt

Based on the installed variant:

| Installed variant | Prompt to use |
|---|---|
| minimal | `full-review.md` (reduced — skip the semgrep/codeql/owasp steps) |
| sec-review | `full-review.md` |
| threat-modeling | `threat-model.md` |
| full | `full-review.md` followed by `threat-model.md` (offer both) |

Prompts are in `.security-kit/prompts/` of the project (copied there by `install`).

### Step 3 — Load and execute the prompt

Read the prompt template. Substitute `{{TARGET}}` with the current working directory (or `$1` if the user gave a different path).

**Always append the F4 gap-closer** from `.security-kit/prompts/shell-string-audit.md`. This is non-negotiable — every adversarial skill misses template-literal shell-command builders without an explicit prompt hint.

### Step 4 — Execute as you would any security review

Work through the prompt stages. Call the installed skills in order (Skill tool invocations). Produce a final report to `.security-kit/scan-<timestamp>.md`.

### Step 5 — Summarize

Print:
- Count of findings by severity
- Path to full report
- Any skills that failed to load (workspace-wiring issues)

## Raptor mode

If the user passes `--with-raptor` and raptor is installed in this project:

1. Hand off to `/raptor-scan $TARGET` instead of the multi-skill chain.
2. Raptor's pipeline will produce `out/scan_<timestamp>/` with SARIF findings, attack traces, PoCs, and unified-diff patches.
3. Follow up with the T6 chain against raptor's SARIF to cross-check.

If `--with-raptor` is set but raptor is not installed, tell the user to run `/sec-kit install-raptor` first.

## Notes

- If `$TARGET` is not provided, scan the current working directory.
- Large codebases (>50k LOC): warn about token cost. Offer to scope to specific subdirs.
- If the user has an existing threat model (`.threatmodel/`), offer to run `tm-drift` instead of the full `tm-full` re-run.
