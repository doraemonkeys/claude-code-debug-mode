# Claude Code Debug Mode

A hypothesis-driven debugging skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Instead of blindly guessing fixes, it instruments your code with runtime logs, generates multiple hypotheses, and iteratively narrows down the root cause — with you in the loop.

Inspired by [Cursor's Debug Mode](https://www.cursor.com/blog/introducing-debug-mode).

## How It Works

```
Bug Report → Hypotheses → Instrument Code → Reproduce → Analyze Logs → Fix → Verify → Clean Up
```

1. **Understand** — Gathers context about the bug (expected vs actual behavior, repro steps)
2. **Hypothesize** — Generates 3–5 testable hypotheses about root causes
3. **Instrument** — Adds targeted debug logging wrapped in `#region DEBUG` blocks
4. **Reproduce** — You trigger the bug while logs are collected to `.claude/debug.log`
5. **Diagnose** — Maps log output to hypotheses, confirms or rules out each one
6. **Fix** — Writes a minimal, targeted fix (not a refactor)
7. **Verify** — You confirm the fix works; if not, the cycle repeats
8. **Clean up** — Removes all instrumentation, leaving a clean diff

## Install

### One-liner (recommended)

```bash
# macOS / Linux
mkdir -p ~/.claude/skills && git clone https://github.com/doraemonkeys/claude-code-debug-mode.git ~/.claude/skills/claude-code-debug-mode

# Windows (Git Bash / MSYS2)
mkdir -p ~/.claude/skills && git clone https://github.com/doraemonkeys/claude-code-debug-mode.git ~/.claude/skills/claude-code-debug-mode
```

### Manual

1. Clone or download this repository into your Claude Code skills directory:

   ```bash
   cd ~/.claude/skills
   git clone https://github.com/doraemonkeys/claude-code-debug-mode.git
   ```

2. That's it. Claude Code automatically discovers skills under `~/.claude/skills/`.

### Verify installation

```bash
ls ~/.claude/skills/claude-code-debug-mode/debug-mode/SKILL.md
```

If the file exists, you're good to go.

## Usage

In Claude Code, simply describe a bug and the skill will activate automatically. You can also invoke it explicitly:

```
/debug-mode Something isn't working — the API returns 200 but the data is empty
```

Or just describe the problem naturally:

```
> The login page shows a blank screen after clicking submit. No errors in the console.
```

Claude will follow the structured debug workflow — generating hypotheses, adding instrumentation, and asking you to reproduce the bug at each step.

## Key Design Decisions

- **Logs go to `.claude/debug.log`** (absolute path), not stdout/stderr — keeps your terminal clean and avoids context window flooding
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
rm -rf ~/.claude/skills/claude-code-debug-mode
```

## License

MIT
