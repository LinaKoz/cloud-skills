---
name: pr-review-council
description: >
  Run a rigorous, multi-angle pull request or code-diff review by spawning three parallel
  subagents, each focused on one risk lens: concurrency & shutdown correctness, auth & secrets
  handling, and cost & reliability. Every subagent cites file:line for each finding; results are
  deduplicated and ranked by severity into one combined report. Use whenever the user asks to
  review a PR, review a pull request, review a diff or branch, wants a "thorough", "deep", "red
  team", or "multi-angle" review of code changes, or explicitly asks for parallel agents / a
  panel / a council to review code — even without naming the three categories. Trigger on "review
  this PR", "review my PR", "check this pull request", "audit this diff", "give this branch a
  thorough review", "use multiple agents to review X". Skip for style/lint-only requests or a
  quick single-file skim with no security/concurrency/cost angle.
---

# PR Review Council

Three independent reviewers catch more than one generalist skimming a diff once. Each subagent
below only looks through its own lens, so it doesn't dilute attention across everything at once
— that's what makes the combined result sharper than a single pass.

## Step 1 — Get the diff

Figure out what's being reviewed, in order of preference:
- A PR URL or `owner/repo#123` reference → use `gh pr diff <number>` (via the shell) or fetch it.
- A local branch → `git diff main...HEAD` (or whatever the user's base branch is) via the shell.
- Pasted code / a diff already in the conversation → use it directly.

If none of these apply, ask the user for the PR link, branch name, or diff before proceeding.
Do not guess at a diff that hasn't been shown to you.

## Step 2 — Spawn exactly three parallel agents

Launch all three Agent tool calls in the **same message** so they run concurrently — this is the
whole point of the exercise, and spawning them one at a time defeats it. Each agent's prompt
must be self-contained (it has no memory of this conversation) and should include: the actual
diff/PR content or exact instructions for retrieving it, its one assigned lens, and the citation
requirement below.

Give each agent one of these lenses:

**Agent 1 — Concurrency & shutdown correctness.** Races, deadlocks, improper locking or lock
ordering, goroutine/thread/task leaks, graceful shutdown and cleanup handling, ordering
assumptions between async operations, idempotency under retries or duplicate delivery.

**Agent 2 — Auth & secrets handling.** Missing or incorrect authn/authz checks, privilege
escalation paths, hardcoded secrets/keys/tokens, secrets that get logged or exposed in
errors/responses, insecure token storage or transmission, missing validation at trust
boundaries.

**Agent 3 — Cost & reliability.** Unbounded loops or retries, missing timeouts or circuit
breakers, N+1 queries or other quadratic-ish hot paths, expensive operations run too often,
resource leaks, error handling/backoff correctness, blast radius if this code fails.

Every agent's prompt must instruct it to:
- Cite `file:line` for every finding — no vague "somewhere in this function" claims.
- Assign each finding a severity: critical / high / medium / low.
- List only real issues it's confident about — not padding with praise or nitpicks dressed up as
  findings.

## Step 3 — Dedupe and rank

Once all three return, merge the findings into a single list:
- Collapse findings that describe the same underlying issue even if flagged by more than one
  agent (this happens at the boundary of two lenses, e.g. an auth check missing inside a retry
  loop). Note when a finding was independently caught by 2+ agents — that's a stronger signal it
  matters.
- Sort the combined list by severity, critical first, then high, medium, low.
- For each entry, show: file:line, category (concurrency / auth / cost-reliability), severity,
  and a one- or two-sentence description of the issue and why it matters.

## Step 4 — Present

Give the user the combined, ranked report. Skip a lengthy preamble — the ranked list is the
deliverable. If any lens came back with nothing, say so briefly rather than omitting it silently
(an empty auth review is itself a useful signal).
