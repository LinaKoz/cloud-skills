---
name: harsh-review
description: Adversarial, verified code review of uncommitted changes, a commit, a branch, a PR (URL or owner/repo#N), or a pasted diff. Covers correctness & concurrency, security, reliability & cost, and API contract & test coverage; every finding cites file:line with a concrete failure scenario and is checked against the real code before it is reported. Three levels, quick (interviews), standard (default, adaptive parallel agents) and deep (always parallel agents). Review-only unless fixes are asked for. Trigger on /harsh-review, "harsh/brutal/no-mercy review", "tear this diff apart", "review this PR", "review my PR", "audit this diff", "thorough/deep/multi-angle/red-team review", or requests for parallel agents, a panel or a council to review code. Skip style/lint-only requests.
---

# Harsh Review

A critical, adversarial review. The goal is to find real bugs, not to validate the diff. Skip praise entirely. Every reported finding must survive a verification check; unverified guesses are dropped, not padded in.

## 0. Pick the level

| Level | When | Parallel agents | Verification |
|---|---|---|---|
| **quick** | user says "quick", "fast", "I'm in an interview", or time is clearly tight | never | code-trace only; unreproduced findings are `PLAUSIBLE` |
| **standard** (default) | anything else | only if the diff is large or risky (see Step 2) | reproduce critical/high findings (time-boxed); trace the rest |
| **deep** | user says "deep", "thorough", "council", "panel", "parallel agents", "multi-angle", or "red team" | always, one per angle | reproduce every finding where feasible |

State the chosen level in one line at the start of the output. If the user names a level, use it.

## 1. Determine the target

In order of preference:
- A PR URL or `owner/repo#123` → `gh pr diff <n>` (and `gh pr view <n>` for the description). If `gh` is unavailable or unauthenticated, ask for the diff or a local branch instead of guessing.
- A named commit / branch / file → `git show <sha>`, `git diff <base>...<branch>`, or that file's diff.
- Nothing named → the uncommitted working tree (`git diff HEAD`, plus `git status` for untracked files). If the tree is clean, review the latest commit (`git show HEAD`).
- A diff or code pasted into the conversation → use it directly.

Never review a diff you have not actually seen. If there is no repo access (pasted diff only), say that verification is limited to reasoning over the provided code.

## 2. Find candidate issues

Read the actual diff **and the surrounding code it touches**: callers, the functions it modifies, related config and tests. Race conditions, auth gaps and contract breaks are often visible only from how the change interacts with code around it.

The four angles:

**A. Correctness & concurrency**: race conditions, lock ordering and deadlocks, ordering assumptions between async operations, task/thread/goroutine leaks, error and graceful-shutdown paths, resource cleanup, retry/idempotency under duplicate delivery, off-by-one and boundary conditions.

**B. Security**: hardcoded or seeded secrets/keys/tokens, secrets in logs, errors or responses, missing or wrong authn/authz on new paths, privilege escalation, injection (SQL/command/template), unsafe defaults (wildcards, permissive CORS, disabled TLS verification), missing validation at trust boundaries, data leaking across a trust boundary.

**C. Reliability & cost**: unbounded growth (caches, queues, connections without eviction or TTL), unbounded loops or retries, retry storms, missing timeouts or circuit breakers on external calls, N+1 queries and hot-path cost, handles/sockets that outlive their owner, blast radius if this code fails.

**D. API contract & tests**: status codes and error shapes, backward compatibility of changed request/response/event shapes, error-path coverage, and whether tests actually exercise the changed behavior or only the happy path.

**Inline vs parallel:**
- **quick**: review all angles yourself in one pass.
- **standard**: review inline unless the diff is large (roughly 5+ files, or any file with 200+ changed lines) **or** risky (touches locking, auth, payments, migrations, or shared infrastructure, where understanding it requires reading a lot of surrounding code). Then use parallel agents as described below.
- **deep**: always use parallel agents.

**Parallel agents** (when an agent/subagent tool is available): launch all agents **in the same message** so they run concurrently: one per angle A-D (or A-C plus D folded into the main agent for a mid-size diff). Use a read-only type such as `Explore` if one exists. Each agent prompt must be self-contained (the agent has no memory of this conversation) and include:
- the diff, or exact commands to obtain it, plus the repo path
- its single assigned angle, with the checklist above
- instructions to read surrounding code, not only changed lines
- output format: for each candidate, give `file:line`, severity (critical/high/medium/low), a concrete failure scenario, and the evidence it looked at
- "report only issues you believe are real; no praise, no style nitpicks; say explicitly if you found nothing"

Agents find candidates. They do not do the final verification; the main agent does (Step 4), because it sees all angles together.

If no agent tool is available, review the angles sequentially yourself, one full pass per angle.

## 3. Merge and dedupe

Before verifying anything:
- Collapse candidates that describe the same underlying issue, even when found from different angles (e.g. an auth check missing inside a retry loop). Keep the clearest description.
- Note when two or more agents found an issue independently; this is a stronger signal.
- Keep track of angles that returned nothing.

Deduping first avoids reproducing the same bug twice.

## 4. Verify every candidate

Static reading alone is not enough. For each candidate, cheapest check first:

1. **Trace**: follow the exact runtime path through the code. If the bug depends on library/framework/SDK behavior, read that source or type signature rather than assuming.
2. **Reproduce** (when the level calls for it and it is cheap enough): write a failing test, a throwaway script, or run the existing test that should catch it. Put throwaway files outside the diff and remove them afterward.
3. **Time-box**: if a reproduction is not converging within a few minutes, stop and mark the finding `PLAUSIBLE`.

Label each surviving finding:
- `CONFIRMED`: reproduced, or traced with certainty through concrete code.
- `PLAUSIBLE`: reasoning is strong from the code, but it was not reproduced (e.g. timing-dependent races).

**Drop** anything that does not survive the check. Never claim a finding was reproduced unless it actually was.

## 5. Report

Rank findings by severity: critical → high → medium → low (`CONFIRMED` before `PLAUSIBLE` within the same severity).

Each finding needs:
- `file:line`
- angle (correctness-concurrency / security / reliability-cost / api-tests)
- severity and verdict (`CONFIRMED` / `PLAUSIBLE`)
- a concrete failure scenario: what input or state triggers it and what breaks
- one line of evidence (test output, traced path)
- "found by N agents" when applicable

Delivery:
- If a `ReportFindings` tool is available, call it **once** with the verified findings, most severe first (empty list if none survived), and do not also print them as text.
- Otherwise, present the ranked list directly, with no long preamble.

Also state briefly: the level used, which angles found nothing (an empty security pass is a useful signal), and anything that could not be verified and why (no repo access, no test runner, flaky environment).

## 6. Fixes (only if asked)

**Do not edit code**: this skill is review-only. Only if the request explicitly asked for fixes too ("review and fix", "find and fix"):
1. Report first (Step 5).
2. Fix each `CONFIRMED` finding with the smallest correct change.
3. Re-run the relevant tests and type checker; never claim they passed without running them.
4. Report again with an outcome on each finding: `fixed` / `skipped` (with reason) / `no_change_needed`. With `ReportFindings`, re-call it with `outcome` set.

## Hard rules

- No praise, no padding, no style nitpicks presented as findings.
- Do not fabricate files, lines, APIs, test results, or command output.
- Every finding cites `file:line` and a concrete failure scenario.
- Unverified findings are dropped; `PLAUSIBLE` is for strong reasoning that could not be reproduced, not for guesses.
- Do not edit code unless fixes were explicitly requested.
