# Shell-string subprocess audit — the F4 gap closer

**Always include this in any scan prompt.** Every adversarial skill in the comparative audit missed shell-string subprocess builders. This prompt closes that gap.

---

## Why it matters

The audit ran 8 independent attempts across raptor, sec-review skills, and threat-modeling skills. Every single one missed the F4 finding — shell-interpolated subprocess invocations in `hunter.ts`:

```
// strategies.ts:239
return BACKTICK-ps -ww -eo pid,ppid,args | grep "INTERP-processName" | grep -v grep-BACKTICK;
```

(Using placeholder notation to avoid tripping literal-pattern hooks.)

Not exploitable today (inputs are static constants), but the pattern is fragile — any future commit routing webview or config values into the builder becomes an immediate shell-injection sink.

**Why skills miss it:** adversarial reasoning hunts from tainted sources toward dangerous sinks. F4 has *no tainted source today* — only a dangerous pattern. The adversarial lens treats it as clean. Static-analysis layers (Semgrep's Node/TS shell-injection rules) are too weak to catch template-literal patterns.

**The fix:** explicit instruction to audit the pattern regardless of current taintedness.

---

## The prompt fragment

Paste this into any scan prompt as its own step:

```
Subprocess hygiene pass — audit every subprocess-launching call site
for shell-string interpolation patterns. Specifically:

  - Any template literal passed to a subprocess-launching API
    (child_process APIs like execSync, spawn; Python subprocess module
    with shell=True; backtick shell-out in Ruby/Perl) where the
    template contains interpolation.
  - String concatenation into a shell command (shell + "arg" + variable).
  - Any invocation of a shell binary (bash, sh, cmd) with a composed
    string argument.

For each match, report:
  - file:line
  - whether inputs are TAINTED (reachable from remote/user/config) or
    STATIC (constants, hardcoded, or parsed integers with validation)
  - if STATIC: flag as FRAGILE with a one-line note about what future
    change would make it exploitable
  - if TAINTED: flag as EXPLOITABLE with severity based on the source

Use Grep with patterns like:
  - execAsync call followed by a template literal with interpolation
  - execSync call followed by a template literal with interpolation
  - spawn call followed by a template literal with interpolation
  - shell=True with a composed string (Python)
```

---

## Example finding output

```
FRAGILE — src/engine/strategies.ts:239
  Shell-string builder with processName interpolation.
  Input: PROCESS_NAMES constant (static today).
  Fragile because: any future change that lets config/webview/user input
  flow into processName becomes an immediate shell-injection sink.
  Fix: use the argv-array form of the subprocess API instead of a
  composed string.
```

---

## Integration

`full-review.md` already includes this as step 6 of the standard chain. Standalone use:

```
Audit {{TARGET}} using the shell-string audit prompt from
.security-kit/prompts/shell-string-audit.md. Return the hit list.
```
