---
name: Debug Mode
description: Interactive debugging mode that generates hypotheses, instruments code with runtime logs, and iteratively fixes bugs with human-in-the-loop verification. Use when the user reports a bug, says something isn't working, or asks to debug an issue.
---

# Debug Mode

You are in **Debug Mode** — a hypothesis-driven debugging workflow. Do NOT jump to fixes. Follow each phase in order.

---

## Phase 1: Understand the Bug

Ask the user (if not already provided): expected vs actual behavior, reproduction steps, error messages.

Read the relevant source code. Understand the call chain and data flow.

## Phase 2: Generate Hypotheses

Generate **3–5 testable hypotheses** as a numbered list:

```
Based on my analysis, here are my hypotheses:

1. **[Title]** — [What might be wrong and why]
2. **[Title]** — [Explanation]
3. **[Title]** — [Explanation]
```

Include both obvious and non-obvious causes (race conditions, off-by-one, stale closures, type coercion, etc.).

## Phase 3: Instrument the Code

### Log file

All debug logs MUST use **absolute path** `{project_root}/.claude/debug.log` (resolve the project root to an absolute path first, never use a relative path — this avoids issues when the process `cwd` differs from the project root). Before each reproduction:
1. Create the `.claude/` directory if needed
2. **Clear** the debug log

Server-side: use file-append APIs (e.g. `fs.appendFileSync`, `open("a")`), http server, or similar.
Browser-side: add a debug API route in server, use `fetch` POST to write `debug.log`.

**Ensure logging works in all environments** (dev/release), not just dev mode.

### Region markers

ALL instrumentation MUST be wrapped in region blocks for clean removal:

```
// #region DEBUG       (JS/TS/Java/C#/Go/Rust/C/C++)
# #region DEBUG        (Python/Ruby/Shell/YAML)
<!-- #region DEBUG --> (HTML/Vue/Svelte)
-- #region DEBUG       (Lua)

...instrumentation...

// #endregion DEBUG    (matching closer)
```

### Logging rules

- **NEVER use `console.log`、`print` or any stdout/stderr output.** All debug output MUST go to `debug.log` — server-side via file-append, browser-side via `fetch` to a debug API endpoint.
- Log messages include hypothesis number: `[DEBUG H1]`, `[DEBUG H2]`, etc.
- Log variable states, execution paths, timing, decision points
- Be minimal — only what's needed to confirm/rule out each hypothesis

After instrumenting, tell the user to reproduce the bug, then **STOP and wait**.

## Phase 4: Analyze Logs & Diagnose

When the user has reproduced:

1. **Check log file size first** (e.g. `wc -l` or `ls -lh`). If the log is large, use `tail` or `grep "[DEBUG H"` to extract only the relevant lines instead of reading the entire file — avoid flooding the context window.
2. Map logs to hypotheses — determine which are **confirmed** vs **ruled out**
3. Present diagnosis with evidence:

```
## Diagnosis

**Root cause**: [Explanation backed by log evidence]

Evidence:
- [H1] Ruled out — [why]
- [H2] Confirmed — [log evidence]
```

If inconclusive: new hypotheses → more instrumentation → clear log → ask user to reproduce again.

## Phase 5: Generate a Fix

Write a fix. Keep debug instrumentation in place.

Clear `.claude/debug.log`, ask user to verify the fix works, then **STOP and wait**.

## Phase 6: Verify & Clean Up

**If fixed:** Remove all `#region DEBUG` blocks and contents (use Grep to find them), delete `.claude/debug.log`, summarize.

**If NOT fixed:** Read new logs, ask what they observed, return to **Phase 2**, iterate.

---

## Rules

- **Never skip phases.** Instrument and verify even if you think you know the answer.
- **Never remove instrumentation before user confirms the fix.**
- **Never use `console.log`、`print` etc.** All debug output goes to `.claude/debug.log` via file-append only.
- **Always clear the log before each reproduction.**
- **Always wrap instrumentation in `#region DEBUG` blocks.**
- **Always wait for the user** after asking them to reproduce.
