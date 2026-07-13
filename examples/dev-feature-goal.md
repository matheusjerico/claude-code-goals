# Objective
Add per-user rate limiting to POST /api/upload: 100 requests per hour per user, returning
HTTP 429 when exceeded.

# Why
The bug only shows up across real requests, so this needs the integration test against the
middleware, not a unit test.

# Done when
- `npm run lint` exits 0 (show the output)
- `npm test` exits 0, including a new test asserting the 101st request in an hour returns 429
- `npm run build` exits 0
- the change is committed with a descriptive message
A green run of all three, surfaced in the transcript, is the proof. Do not claim done without it.

# Out of scope
- Do not change the auth middleware or the upload handler's business logic.
- Use the rate limiter already in package.json; add no new dependency.
- Never edit, weaken, skip, or delete a test to make the suite pass.

# If it fails
- If the integration test is not green within the budget, open a draft PR with what you have
  and a note on what is blocking. Do not force it.
- STOP CONDITION: 15 turns or $2, whichever comes first.
