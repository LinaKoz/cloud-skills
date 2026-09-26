---
name: harsh-review
description: Multi-angle, empirically-verified critical review of uncommitted changes, a specific commit, or a diff. Reviews for concurrency/shutdown/race correctness, security (secrets, auth, injection, unsafe defaults), reliability/cost (leaks, unbounded growth, missing timeouts), and API-contract/test-coverage gaps — then verifies every candidate finding against the live repo before reporting it, rather than trusting static analysis alone. Reports via ReportFindings, ranked by severity, and does not edit code unless the invocation explicitly asks for fixes too. Trigger: /harsh-review, "harsh review", "brutal review", "tear this diff apart", "no-mercy review".
---

# Harsh Review

A critical, adversarial review — the goal is to find real bugs, not to validate the diff. Skip praise entirely.

## 1. Determine the target

Default to the uncommitted working tree diff (`git diff HEAD` / `git status`). If the user names a commit, PR, or file, review that instead. If the tree is clean and nothing else was named, review the most recent commit (`git show HEAD`).

## 2. Review from every angle below

Read the actual diff and the surrounding code it touches — not just the changed lines in isolation; a race condition or auth gap is often visible only from how the change interacts with code around it.

For a diff spanning many files (roughly 5+ touched files, or any file over ~200 changed lines), split the angles across parallel `Agent` calls (subagent_type `Explore` for read-only investigation, or a fresh review pass per angle) so each angle gets full attention instead of being rushed. For a small diff, do all angles yourself inline — spawning agents for a 20-line diff is wasted overhead.

**Correctness & concurrency**: race conditions, ordering assumptions between async operations, error/shutdown paths, resource cleanup, retry/idempotency, off-by-one and boundary conditions.

**Security**: hardcoded or seeded secrets/API keys, missing auth on new endpoints, injection (SQL/command/template), unsafe defaults (wildcard keys, permissive CORS, disabled verification), data leaking across a trust boundary (e.g. an internal error message reaching a client response).

**Reliability & cost**: unbounded growth (caches/queues/connections with no eviction or TTL), missing timeouts on external calls, N+1 queries, handles/sockets that can outlive their owner, retry storms.

**API/contract correctness**: status codes, error-path coverage, backward compatibility of changed shapes, and — critically — whether tests actually cover the changed behavior or just the happy path.

## 3. Verify empirically before reporting anything

Do not trust static analysis. For every candidate finding, before it counts as real:

- Reproduce it — write a quick failing test, a throwaway script, or trace the exact runtime path with certainty (read the actual library/framework source if the bug depends on its behavior, the way a type assumption or an SDK method's real signature can be wrong).
- If you can't reproduce it but the reasoning is airtight from reading the code, mark it `PLAUSIBLE` rather than `CONFIRMED`.
- If it doesn't survive this check, drop it. Don't pad the finding list with unverified static-analysis guesses.

Dedupe overlapping findings from different angles/agents before reporting.

## 4. Report

Call `ReportFindings` once with the verified findings, most severe first. Each finding needs a concrete failure scenario (what input/state triggers it, what breaks), not just a description of the pattern.

**Do not edit any code** — this skill is review-only. If the user's invocation explicitly asked for fixes too (e.g. "review and fix", "find and fix"), implement the fix for each `CONFIRMED` finding after reporting, re-run the test suite and type checker, and re-call `ReportFindings` with `outcome` set on each finding (`fixed` / `skipped` / `no_change_needed`).
