# Threat model — STRIDE workflow

Full STRIDE threat model using josemlopez/threat-modeling-toolkit. Produces machine-readable state files (`.threatmodel/state/*.json`), Mermaid diagrams, and two markdown reports.

---

## The prompt

```
Run the tm-init → tm-threats → tm-report workflow on {{TARGET}}.

Identify assets, actors, trust boundaries, and STRIDE threats per boundary.
Map each threat to OWASP/CWE. Don't skip threats that look "by design" — I
want a full picture including intentional-but-risky features.

After the main workflow:
  - Run tm-verify to cross-check the threat-to-mitigation mapping.
  - Run tm-compliance to map threats to relevant compliance frameworks
    (SOC2, ISO-27001, etc.) if the project is customer-facing.

Write a baseline snapshot to .threatmodel/baseline/snapshot-$(date +%Y%m%d).json
so future runs of tm-drift can detect changes.

Output:
  - Executive summary (.threatmodel/reports/executive-summary.md)
  - Full risk report (.threatmodel/reports/risk-report.md) with STRIDE
    + OWASP + CWE breakdown, severity, and 3-phase mitigation roadmap.
  - Baseline snapshot for drift detection.
```

---

## Substitutions

- `{{TARGET}}` — project path

---

## Notes

- **Coverage:** this produces ~20-25 threats for a medium codebase. Expect 4-6 CRITICAL/HIGH, rest MEDIUM/LOW with a healthy "by-design flagged" section.
- **Cost:** ~30-50k tokens for a 15k-LOC project. Scales roughly linearly with codebase size.
- **Output location:** the toolkit writes into `.threatmodel/` at the project root. If that directory exists, tm-init asks before overwriting.
- **Re-runs:** after initial run, use `tm-drift` (via `pr-diff-triage.md`) to detect changes instead of re-running the full chain. Full re-run only when the architecture has changed materially.
