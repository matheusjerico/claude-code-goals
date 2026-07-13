# Writing Goals for Claude Code

A practical reference for writing `/goal` commands and autonomous loops with Claude Code. Companion to the article *Stop Prompting Claude Code. Write the Loop That Prompts It.*

A good goal describes the destination precisely enough that the agent picks its own route, and lets both of you verify when it has arrived. It has five sections.

## Cheat sheet

```
# Objective     name the feature or outcome; self-contained, survives a context reset
# Why           only if it changes the plan (delete test)
# Done when      observations not activities; one fact per line; never "or"; show the evidence
# Out of scope   forbid the worst misinterpretation; never edit/weaken/skip a test or the gate
# If it fails    a recovery line + a STOP CONDITION (turns, time, or cost) on every goal
```

Copy the blank skeleton from [`templates/loop-goal.md`](templates/loop-goal.md). Filled examples are in [`examples/`](examples/).

## The five sections, with the rules that matter

### 1. Objective
Name the feature or outcome, not the action. Make it self-contained, so it survives a context reset, a compaction, or a handoff to another session. Write it as if the agent knows nothing about today's chat.

- Good: "Deploy the rate-limit change (PR #123, 100 uploads/hour/user) to production"
- Bad: "deploy to production" (depends on conversation memory)

### 2. Why
Include it only when it disambiguates. The delete test: if removing the Why would not change the agent's plan, delete it. A good Why kills a class of cheap interpretations ("needs testing with real data" rules out local emulators and unit tests).

### 3. Done when
Write observations, not activities. "Test the reaction" is an activity the agent can claim it did; "the bot posts a rocket reaction on the test PR" is an observation that is either true or false.

- One fact per line. Compound criteria hide partial failures.
- Never "or." The agent will take the cheaper branch.
- Order weakest to strongest, so the last line is the one that actually proves the goal.
- Run the check and show the output. The `/goal` judge is a small, lenient model that reads only the transcript, so a claim is not proof. For anything that matters, a deterministic gate (a Stop hook or CI running the check) is the real proof, and it lives outside the goal.

### 4. Out of scope
The highest-leverage section, because it shapes decisions before they are made instead of catching mistakes after. Ask: what is the worst reasonable misinterpretation of my objective, and forbid it explicitly.

- Protect the oracle: never edit, weaken, skip, or delete a test or the gate to make it pass. The cheapest way to turn a red gate green is to break the gate.

### 5. If it fails
The sad path. One line of recovery removes the most dangerous ambiguity: an agent improvising recovery under failure. Mandatory for irreversible or production work.

- STOP CONDITION on every goal: a completion bound (turns, time, or cost), so the loop halts instead of looping forever.

For monitoring or verification goals, the deliverable is a verdict plus evidence, not a code change.

## How to run it

Full detail in [`running-in-claude-code.md`](running-in-claude-code.md). Short version:

```bash
claude --append-system-prompt-file .claude/loop-goal.md
```
then, inside the session, with Auto mode on:
```
/goal <your Done When condition>, or stop after 20 turns
```

Three ways to run the loop, by the shape of the work:
- **`/goal`** (model-judged) for a stable goal you state.
- **Stop hook** (deterministic gate) for work with a clear pass/fail signal: iterate until green.
- **Headless `claude -p` loop** for unattended, multi-context runs that must survive context resets.

## Why five sections and these rules

The structure is validated against the current state of the art: Anthropic's `/goal` docs (one measurable end state, a stated check, constraints that matter), and the agent-goal literature (observable/atomic/bounded acceptance criteria, separating the doer from the done-checker, protecting the gate from reward hacking, always capping the loop).

Written by Matheus Jericó.
