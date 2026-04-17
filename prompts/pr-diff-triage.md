# PR diff triage — does this change warrant a threat-model update?

Chains `differential-review` (what changed) with `tm-drift` (does the TM still hold). This is the closest thing to a "benign vs significant" PR triage tool today.

Prerequisite: the project has an existing threat model in `.threatmodel/` (created via `/security-kit scan` with the threat-modeling variant, or `tm-full` directly). If not, this prompt falls back to differential-review alone.

---

## The prompt

```
PR triage for {{TARGET}}.
Base: {{BASE_REF}}  Head: {{HEAD_REF}}  ({{COMMIT_COUNT}} commits)

Step 1 — differential-review. Classify each commit in the range by:
  (1) blast radius (files touched, LOC changed, modules affected)
  (2) trust boundaries crossed (auth, IPC, remote fetches, webview
      messaging, subprocess calls, file I/O outside workspace)
  (3) new attack surface introduced
Produce a triage table: commit | files | blast | boundaries | new surface.

Step 2 — for commits flagged HIGH in step 1, use tm-drift against the
current threat model snapshot at .threatmodel/baseline/. For each flagged
commit, classify as:
  - benign (no TM update needed)
  - update existing threats (list which threat IDs)
  - requires new threats (list what they would cover)
Explain the reasoning per commit.

Step 3 — final verdict. One of:
  (a) merge clean — no TM changes needed
  (b) merge after TM update — list the specific TM edits to make
  (c) requires TM re-run before merge — list why (major architectural
      changes, new trust boundaries, new external inputs)

Output format:
  - Commit-by-commit triage table
  - TM impact per HIGH commit
  - Verdict + action items
```

---

## Substitutions

- `{{TARGET}}` — project path
- `{{BASE_REF}}` — base branch/ref (default: `main`)
- `{{HEAD_REF}}` — PR branch/ref (default: `HEAD`)
- `{{COMMIT_COUNT}}` — number of commits in the range (auto-detect via `git rev-list --count`)

---

## Notes

- **tm-drift is state-based, not diff-based.** It compares `.threatmodel/state/` against `.threatmodel/baseline/` — the baseline snapshot captured when the threat model was last audited. It's not git-aware. This prompt bridges the two by running differential-review to identify what changed, then tm-drift to verify TM coverage of the changed areas.
- **Heuristic for step 2:** not every HIGH commit invalidates the TM. A HIGH-blast-radius commit that strictly extends existing functionality (e.g., adds a new command ID under an existing allowlist) usually `update existing threats`. A HIGH commit that introduces a new trust boundary (new IPC channel, new external input) usually `requires new threats`.
- **Falls back gracefully:** if no `.threatmodel/` exists, the prompt does only differential-review and recommends creating a baseline via `tm-init` + `tm-full`.
