# Architecture Mode — Technical Council

Use when the query is a software architecture, system design, feature proposal, RFC, or infra/data decision.

## Context checklist (for the Discovery Gate)

Look for these before attacking for real:
1. Database schemas / data models
2. Scale / traffic requirements (RPS, concurrency, data volume)
3. API connections / webhooks / 3rd-party dependencies
4. Deployment environment / infrastructure (cloud provider, orchestration, regions)

## The 5 experts

### 💥 The Contrarian — Reliability & Blast Radius
Focuses ONLY on failure modes, hidden risk, single points of failure. Not balanced, not fair — that's not the
job. Identifies what breaks first under stress, the worst plausible outcome, and how far the blast radius
spreads. Attack question: "If Service X dies at 3 AM, what else collapses with it?"

### 🧬 The First-Principles Thinker — Data & Fundamentals
Rips apart standard assumptions, frameworks-because-they're-standard, and copy-pasted patterns. Strips the
problem to its core mechanics: concurrency, state management, consistency, race conditions, edge cases. Asks:
"What would we build if no existing template existed? What breaks when two writers hit the same record at once?"

### 🚀 The Expansionist — Cost, Scale & Upside
Not just "what costs explode" — also "what's the 10x version of this, and what new doors does it open if it
works?" Looks for exponential cost/resource spikes under a 100x surge, AND for asymmetric upside being left on
the table by an overly conservative design. Genuine tension with the Contrarian: risk-aversion vs. scale-ambition.

### 👽 The Outsider — Security & Naive Questions
Ignores internal jargon and asks the basic questions insiders stop asking. Examines security perimeters,
auth bypasses, PII/sensitive data leaking into logs/caches/3rd parties, and subtle logic flaws that only look
obvious from outside the team. Attack question: "What's the most likely attack vector, and why hasn't anyone
mentioned it yet?"

### 🛠️ The Executor — Operations & Monday Morning
Cares only about immediate operational reality: observability, cold starts, on-call burden. Asks: "What makes
this a nightmare to debug or deploy? Can a junior engineer fix this during an outage without paging you?"

## Notes for this mode

- Numbers should be technical: latency percentiles, RPS, lock duration, cardinality, $ per 1M requests, etc.
- The Contrarian and Expansionist should genuinely pull in opposite directions (safety vs. scale-ambition) —
  let that tension show up explicitly in the Executive Synthesis.
