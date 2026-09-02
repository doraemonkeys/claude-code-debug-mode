# Debug Mode Skill (Claude Code + Codex + Gemini CLI)

A hypothesis-driven debugging skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Codex, and [Gemini CLI](https://github.com/google-gemini/gemini-cli). Instead of blindly guessing fixes, it instruments your code with runtime logs, generates multiple hypotheses, and iteratively narrows down the root cause — with you in the loop.

Inspired by [Cursor's Debug Mode](https://www.cursor.com/blog/introducing-debug-mode).

## How It Works

```
Bug Report → Hypotheses → Instrument Code → Reproduce → Analyze Logs → Fix → Verify → Clean Up
```

1. **Understand** — Gathers context about the bug (expected vs actual behavior, repro steps)
2. **Hypothesize** — Generates 3–5 testable hypotheses about root causes
3. **Instrument** — Adds targeted debug logging wrapped in `#region DEBUG` blocks
4. **Reproduce** — You trigger the bug while logs are collected to `.agents/debug.log`
5. **Diagnose** — Maps log output to hypotheses, confirms or rules out each one
6. **Fix** — Writes a minimal, targeted fix (not a refactor)
7. **Verify** — You confirm the fix works; if not, the cycle repeats
8. **Clean up** — Removes all instrumentation, leaving a clean diff

## npx One-liner Install

```bash
npx skills add https://github.com/doraemonkeys/claude-code-debug-mode --skill debug-mode -g
```

### One-liner (Claude Code)

```bash
# macOS / Linux / Windows (Git Bash / MSYS2)
mkdir -p ~/.claude/skills && git clone https://github.com/doraemonkeys/claude-code-debug-mode.git ~/.claude/skills/claude-code-debug-mode

# Claude Code discovers skills only ONE directory level under ~/.claude/skills/
# (it does not recurse), but this repo keeps SKILL.md inside debug-mode/. Lift that
# folder up one level, otherwise the skill silently won't load:
mv ~/.claude/skills/claude-code-debug-mode/debug-mode ~/.claude/skills/debug-mode
rm -rf ~/.claude/skills/claude-code-debug-mode
```

> **Why the extra step (Claude only):** Codex and Gemini scan `skills/` recursively, so
> they find `claude-code-debug-mode/debug-mode/SKILL.md` on their own. Claude Code looks
> only at the immediate children of `~/.claude/skills/`, so `SKILL.md` must sit exactly one
> level deep — `~/.claude/skills/debug-mode/SKILL.md`. With the nested path it loads no
> skill and reports no error, so the gap is easy to miss.

### One-liner (Codex)

```bash
# macOS / Linux / Windows (Git Bash / MSYS2)
mkdir -p ~/.codex/skills && git clone https://github.com/doraemonkeys/claude-code-debug-mode.git ~/.codex/skills/claude-code-debug-mode
```

### One-liner (Gemini CLI)

```bash
# macOS / Linux / Windows (Git Bash / MSYS2)
mkdir -p ~/.gemini/skills && git clone https://github.com/doraemonkeys/claude-code-debug-mode.git ~/.gemini/skills/claude-code-debug-mode
```

Or use the built-in install command:

```bash
gemini skills install https://github.com/doraemonkeys/claude-code-debug-mode.git
```

### Manual

1. Clone or download this repository into your skills directory:

   ```bash
   # Claude Code
   cd ~/.claude/skills
   # Codex
   cd ~/.codex/skills
   # Gemini CLI
   cd ~/.gemini/skills

   git clone https://github.com/doraemonkeys/claude-code-debug-mode.git
   ```

2. That's it. Skills are automatically discovered under `~/.claude/skills/`, `~/.codex/skills/`, and `~/.gemini/skills/`.

   For **Claude Code**, apply the same lift step as the one-liner above — move the inner `debug-mode/` folder up so `SKILL.md` lands at `~/.claude/skills/debug-mode/SKILL.md`. Codex and Gemini need no extra step.

### Verify installation

```bash
# Claude Code
ls ~/.claude/skills/debug-mode/SKILL.md

# Codex
ls ~/.codex/skills/claude-code-debug-mode/debug-mode/SKILL.md

# Gemini CLI
ls ~/.gemini/skills/claude-code-debug-mode/debug-mode/SKILL.md
# or
gemini skills list
```

If the file exists, you're good to go.

## Usage

In Claude Code, Codex, or Gemini CLI, simply describe a bug and the skill will activate automatically. You can also invoke it explicitly:

```
/debug-mode Something isn't working — the API returns 200 but the data is empty
```

Or just describe the problem naturally:

```
> The login page shows a blank screen after clicking submit. No errors in the console.
```

The agent will follow the structured debug workflow — generating hypotheses, adding instrumentation, and asking you to reproduce the bug at each step.

## Don't Be Too Rigid — It's About Evidence, Not the Recipe

The skill describes one concrete path (write logs to a local `.agents/debug.log` via an absolute path), and that default works for **most** situations. But the workflow isn't a strict ritual — the real core is always the same two things:

1. **Reproduce** the bug.
2. **Collect evidence** (logs) from that reproduction and hand it back to the agent.

*How* the evidence gets collected and delivered is flexible. As long as the agent ends up with the logs from a real reproduction, the result is identical. A couple of common adaptations:

- **The default local-file path doesn't fit your runtime.** For example, the code runs on a phone (a mobile app), in a remote device, or anywhere it can't write to your project folder. No problem — have it log to a file *on that device*, reproduce there, then copy or paste those logs back to the agent. The transport changed; the workflow didn't.

- **The bug is intermittent / hard to trigger on demand.** You don't have to reproduce while the agent waits. Let the agent add the debug instrumentation up front, then just keep it in place and go about your work. Whenever the bug finally happens, grab the logs and drop them to the agent for analysis. Prepared-and-waiting works just as well as reproduce-on-the-spot.

When the standard flow fits, follow it. When it doesn't, keep the two core steps and adapt the rest — the agent can work with evidence from any source.

## Key Design Decisions

- **Logs go to `.agents/debug.log`** (absolute path), not stdout/stderr — keeps your terminal clean and avoids context window flooding
- **`#region DEBUG` markers** wrap all instrumentation for reliable, automated cleanup
- **Hypothesis-tagged logs** (`[DEBUG H1]`, `[DEBUG H2]`) map directly back to hypotheses for clear diagnosis
- **Human-in-the-loop** — the agent never removes instrumentation or declares victory until you confirm the fix

## Supported Languages

The `#region DEBUG` markers work with:

| Syntax | Languages |
|---|---|
| `// #region DEBUG` | JavaScript, TypeScript, Java, C#, Go, Rust, C, C++ |
| `# #region DEBUG` | Python, Ruby, Shell, YAML |
| `<!-- #region DEBUG -->` | HTML, Vue, Svelte |
| `-- #region DEBUG` | Lua |

## Uninstall

```bash
# Claude Code
rm -rf ~/.claude/skills/debug-mode

# Codex
rm -rf ~/.codex/skills/claude-code-debug-mode

# Gemini CLI
rm -rf ~/.gemini/skills/claude-code-debug-mode
# or
gemini skills uninstall debug-mode
```

## License

MIT
