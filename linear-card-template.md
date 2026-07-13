# Agent-ready Linear card template

A card an autonomous loop can pick up and implement into a PR without a human in the tick. The rule is simple: a well-refined card is a well-formed goal. If a field would make the agent guess, it is not ready.

The card supplies the per-item parts of the goal (Objective, Why, Done when, item scope). The standing `loop-goal.md` supplies the global parts (out-of-scope defaults, the gate, handoff, stop condition). See `templates/loop-goal.md` and `README.md`.

## Template (copy into the Linear description)

```markdown
## Objective
<verb + outcome, self-contained; readable with zero knowledge of any chat or meeting>

## Context / Why
- Problem: <what is wrong or missing, and why now>
- Links: <design doc, related PR, screenshot, thread>
- Impact: <who or what this affects>

## Acceptance criteria (done when)
<observable facts, one per line, no "or", weakest to strongest>
- [ ] <observable fact 1>
- [ ] <observable fact 2>
- [ ] <the fact that actually proves the outcome, last>
- [ ] Test that proves it: <exact test name or command, e.g. "new test asserts the 101st request in an hour returns 429">

## Scope
- In scope: <files, modules, or areas to change>
- Do NOT touch: <frozen files, public APIs, auth, infra, dependencies>

## Technical notes / pointers
- Entry points: <files, functions, routes>
- Follow the pattern in: <example file or module>
- Approach (optional, non-binding): <hint, if you have one>
- Test data / fixtures: <what to use>

## Verification
- Gate (must be green): <e.g. npm run lint && npm test && npm run build>
- Manual check (if any): <steps to reproduce or confirm by hand>

## Dependencies / preconditions
- Blocked by: <#card, or none>
- Needs: <env var, access, feature flag, seed data>

## If blocked
- Stop and comment on this card if: <ambiguity, missing access, a product decision is required>
```

## Definition of Ready (the gate for the `ready-for-agent` label)

Do not label a card `ready-for-agent` until every box is true. This is the entry gate the loop depends on; it is the agile Definition of Ready, tightened for a machine.

- [ ] Title names a self-contained outcome (no reliance on conversation or tribal knowledge)
- [ ] Every acceptance criterion is observable (a file, response code, log line, test result), not an activity
- [ ] At least one acceptance criterion is backed by a test or command that proves it
- [ ] No "or" in the acceptance criteria
- [ ] In-scope and do-not-touch are both explicit
- [ ] The gate command is named and the repo is green on it today (known-good baseline)
- [ ] No open product questions; assumptions, if any, are written down
- [ ] The card is small enough to finish in one bounded run (split it if not)

If a card fails any box, it goes back to refinement, not to the agent.

## Filled example

```markdown
## Objective
Add per-user rate limiting to POST /api/upload: 100 requests per hour per user, HTTP 429 on exceed.

## Context / Why
- Problem: one client is saturating the upload workers; there is no per-user limit today.
- Links: <incident thread>, <api design doc>
- Impact: all upload users during abuse windows.

## Acceptance criteria (done when)
- [ ] a normal user's first 100 uploads in an hour succeed
- [ ] the 101st upload within the same hour returns HTTP 429 with a clear message
- [ ] the limit is per user, not global (two users do not share a bucket)
- [ ] Test that proves it: new integration test asserts the 101st request returns 429; npm test green

## Scope
- In scope: the upload route and its middleware, plus tests
- Do NOT touch: auth middleware, the upload handler's business logic, dependencies, CI, infra

## Technical notes / pointers
- Entry point: src/routes/upload.ts
- Follow the pattern in: src/middleware/throttle.ts
- Use the rate limiter already in package.json; add no new dependency
- Test data: two seeded test users

## Verification
- Gate (must be green): npm run lint && npm test && npm run build
- Manual check: send 101 requests as one user, confirm the last is 429

## Dependencies / preconditions
- Blocked by: none
- Needs: no new env vars

## If blocked
- Stop and comment if the desired limit window or response body is unclear.
```

## How the loop consumes it

1. The driver finds the next card labeled `ready-for-agent`.
2. It writes the card body into `TASK.md` (the per-tick Objective, Done when, scope, notes).
3. The standing `loop-goal.md` adds the global out-of-scope, the gate, the handoff protocol, and the stop condition.
4. The agent implements the smallest increment, runs the gate, records progress, and repeats until the acceptance criteria hold and the gate is green.
5. On green + done, the loop opens the PR and moves the card to In Review. On blocked, it comments and escalates.

The better the card, the less the agent guesses. Refinement is where the work moved.
