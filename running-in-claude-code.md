# Running a goal loop in Claude Code

The goal (the five sections) is the prose. The loop config (allowed tools, budgets, the gate, handoff, output) lives in flags and hooks. Keep them separate. Claude Code gives you three ways to run the loop; pick by the shape of the work.

| Way | Next turn starts when | Stops when | Use it for |
|-----|----------------------|------------|------------|
| `/goal` | The previous turn finishes | A model judge confirms the condition | A stable goal you state; the model finds the path |
| Stop hook | The previous turn finishes | Your script decides | A clear pass/fail signal (tests, types, lint): iterate until green |
| Headless `claude -p` loop | Your script calls it again | Your script decides | Unattended, multi-context runs that must survive context resets |

## 1. `/goal` (model-judged, you are around)

Put the goal in `.claude/loop-goal.md`, launch the session with it appended (append, not replace, so Claude keeps its coding behavior), and set a `/goal` condition with a bound. Enable Auto mode so turns run unattended.

```bash
claude --append-system-prompt-file .claude/loop-goal.md
```
```
/goal every acceptance criterion is met and `npm run lint && npm test && npm run build` all pass and the work is committed, or stop after 20 turns
```

Subtlety: the `/goal` judge is a small, lenient model reading only the transcript. It does not run your tests. So the goal must make Claude run the checks and print the results, and for anything that matters you should back it with a deterministic gate (below).

## 2. Stop hook (deterministic gate, iterate until green)

`.claude/settings.json`:
```json
{ "hooks": { "Stop": [ { "hooks": [ { "type": "command", "command": "./.claude/check-done.sh" } ] } ] } }
```

`.claude/check-done.sh`:
```bash
#!/usr/bin/env bash
if npm run lint && npm test && npm run build; then
  exit 0   # green: allow stop
fi
echo '{"decision":"block","reason":"Gates are red. Fix the failing lint/test/build, then continue."}'
```

Claude cannot stop while the gate is red. Pair it with `/goal` for the strongest setup: `/goal` decides intent, the hook proves the tests are actually green.

## 3. Headless loop (unattended, survives context resets)

You write the loop once; it prompts Claude Code, you do not. Each `claude -p` is a fresh context; continuity comes from a progress file plus git.

```bash
#!/usr/bin/env bash
set -euo pipefail
MAX=20; i=0
while (( i++ < MAX )); do
  claude -p "Continue per the goal. Read claude-progress.md first, make the smallest next increment." \
    --append-system-prompt-file .claude/loop-goal.md \
    --allowedTools "Read" "Edit" "Write" "Bash(npm *)" "Bash(git *)" \
    --permission-mode acceptEdits \
    --max-turns 30 \
    --output-format json
  if npm run lint && npm test && npm run build; then echo "green, done"; break; fi
done
```

## The loop config, mapped to flags

| Concern | Mechanism |
|---------|-----------|
| Where the goal lives | `CLAUDE.md`, or `.claude/loop-goal.md` via `--append-system-prompt-file` |
| Allowed tools | `--allowedTools`, or `--permission-mode acceptEdits` |
| The gate (real proof) | a Stop hook or CI running `lint`/`test`/`build` |
| Budget / cap | `--max-turns`, the `/goal` bound clause, the loop's iteration cap |
| Progress and handoff | a progress file plus a git commit every turn |
| Output contract | `--output-format json`, or a `STATUS:` last line the driver parses |
