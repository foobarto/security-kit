# Full security review — multi-skill chain

The ground-truth-tested T6 chain from the comparative audit. This prompt consistently produces the most novel findings with the best cost/rigor ratio. Use it as the default.

---

## The prompt

```
Security review of {{TARGET}}.
{{LANGUAGE}} project, approx {{LOC}} lines. {{PROJECT_TYPE_DESCRIPTION}}
Treating as untrusted third-party I'm deciding whether to {{REVIEW_PURPOSE}}.

Use these skills in order:

1. audit-context-building — produce a line-by-line architectural map of the
   attack surface. Enumerate every external input source (remote fetches,
   webview postMessage handlers, IPC sockets, file reads outside the
   workspace, OAuth callbacks, subprocess calls). Don't propose bugs yet —
   just the map.

2. Using that map, hunt for vulnerabilities across all trust boundaries.
   Focus on attack chains reachable from remote sources (config files
   fetched over the network, user input, webview content, any IPC traffic).

3. variant-analysis — for each finding, check whether the same pattern
   appears elsewhere in the codebase. Report variants.

4. supply-chain-risk-auditor — audit package manifests, lockfiles,
   install scripts, and any runtime code that fetches remote content.
   Flag anything that could let the maintainer (or anyone who compromises
   them) push code execution to end users.

5. insecure-defaults — hunt for hardcoded secrets, weak auth, permissive
   defaults, fail-open configs.

6. Subprocess hygiene pass — audit every subprocess-launching call
   site (template-literal shell commands with ${...} interpolation,
   string concatenation into shell strings). Flag fragile patterns even
   if inputs are static today. This is the F4 gap-closer; without this
   explicit instruction every adversarial skill misses the pattern.

7. fp-check — ONLY after you've produced findings, run fp-check on the
   medium-and-below items to eliminate false positives. Do NOT reject
   findings without a concrete reason; if the reason is "the platform
   sandboxes this," verify it's actually true before dropping the finding.

Output: file:path:line references, severity (critical/high/medium/low),
attack scenario in 1-2 sentences, concrete fix in 1-2 sentences. Triage
by severity. List any skill that didn't activate at the end so I can
debug the wiring.
```

---

## Substitutions

- `{{TARGET}}` — absolute path to the project being reviewed
- `{{LANGUAGE}}` — primary language (TypeScript, Python, Rust, etc.)
- `{{LOC}}` — approximate source line count (from `wc -l` on source files)
- `{{PROJECT_TYPE_DESCRIPTION}}` — one sentence, e.g. "VS Code extension that monitors AI quotas", "Node backend serving a REST API", "CLI tool for image processing"
- `{{REVIEW_PURPOSE}}` — "install", "deploy", "merge", "ship", as appropriate

`smart-scan` fills these automatically from the fingerprint.

---

## Why this ordering matters

The order is deliberate:

1. **Map before hunt.** `audit-context-building` runs first to build architectural context. Hunting for bugs before understanding the system produces lots of surface-level findings and misses cross-file chains.
2. **Hunt general, verify specific.** Step 2 hunts broadly across the whole map; steps 3-6 apply specific lenses.
3. **Variants right after hunt.** While the patterns are fresh, check whether each finding recurs elsewhere. Cheap cross-checking.
4. **Static lenses last.** `supply-chain-risk-auditor`, `insecure-defaults`, and the subprocess pass apply well-defined heuristics; they're cheap to run and produce stable output.
5. **fp-check last, with constraints.** fp-check can over-prune without enough context (proven in T5 of the audit). The "don't drop without concrete reason" instruction is load-bearing — don't remove it.

---

## Variations

**Minimal bundle installed:** skip step 6's static-analysis tools (semgrep/codeql); keep the subprocess hygiene pass.

**Threat-modeling bundle installed:** run this prompt first, then `/sec-kit scan` again with `threat-model.md`.

**Raptor installed and you want PoCs:** append:

```
After the skill chain completes, hand off to /raptor-scan {{TARGET}} to
produce runnable exploits and unified-diff patches for any critical or
high severity findings.
```
