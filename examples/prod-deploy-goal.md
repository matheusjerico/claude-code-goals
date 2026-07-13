# Objective
Deploy the rate-limit change (PR #123: max 100 uploads per hour per user, HTTP 429 on
exceed) to production.

# Why
Production still runs the previous release without the limit, and one client is saturating
the upload workers. The fix is merged to main and validated in the sandbox environment.

# Done when
- the release pipeline completes green for the new stable tag
- the API and worker pods run the new tag, Ready, with no restarts
- a smoke request as a normal user succeeds, and the 101st request in an hour returns 429
Show the pipeline run URL, the `kubectl get pods` output, and the smoke-test responses.

# Out of scope
- No changes to autoscaling, timeouts, or cost limits.
- Deploy only via the pipeline; no manual kubectl edits.
- Never edit or skip a test or the gate to make the deploy pass.

# If it fails
- If pods are not healthy within 10 minutes, redeploy the previous stable tag via the
  pipeline and report what blocked.
- STOP CONDITION: 30 minutes, then report status and stop.
