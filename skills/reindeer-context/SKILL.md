---
name: reindeer-context
description: Company and product context for Lina's Reindeer AI interview, where she solves an engineering task with AI. Use whenever Reindeer or Reindeer AI is mentioned, or when working on the Reindeer interview assignment, to understand the business environment behind the task, raise relevant questions, edge cases and trade-offs. Background only; it complements ai-coding-interview and harsh-review and never overrides the actual task requirements or repository.
---

# Reindeer AI — Company and Product Context

Background for the Reindeer interview task. **The actual task requirements and repository always guide implementation.** This context helps identify relevant questions, edge cases and trade-offs; it does not add requirements.

Use it alongside:
- `ai-coding-interview`: the working process (clarify → context → plan → tests → staged implementation → verification).
- `harsh-review`: reviewing the result (quick / standard / deep levels).

## Summary

**Product**: Based on previously reviewed company materials, Reindeer builds enterprise AI agents that execute business processes across existing systems, handling exceptions, incomplete information, legacy integrations, and human involvement.

**Interview observations** (reported by the candidate): workflows may last up to a week, involve humans, and include additional tool calls to gather missing information before making decisions.

**Core challenge**: Make the platform reusable across organizations while adapting to each customer's systems, terminology, policies, permissions, and operational practices. Relevant knowledge may be spread across databases, documents, communications, and human experts.

**Engineering considerations** (design inferences, not confirmed architecture):
- Combine explicit workflows for predictable steps with agent decisions where the next action depends on new evidence.
- Support durable state, waiting for humans, and resuming after failures.
- Retrieve relevant, authorized context; distinguish missing information and failed lookups from business conclusions.
- Enforce permissions and business rules in application logic, and make retries safe through idempotency and reconciliation.
- Keep decisions, evidence, tool outcomes, and human approvals traceable.
- Evaluate changes against representative cases and regressions; measure quality, latency, and cost.

Apply only what is relevant to the assignment and available time. Read the detailed sections below only when the task touches that theme.

## Rules for using this context

- Keep three kinds of statements distinct and label them when it matters: **company claims** (from company materials), **interview observations** (what the candidate was told), and **design assumptions** (our inferences).
- Treat Temporal, LangGraph, graph databases, vector databases, queues, and other technologies discussed during preparation as possible options, not confirmed Reindeer technologies or required choices.
- Do not infer Reindeer's stack from an employee's technical posts.
- When a company-specific detail is unknown, state the assumption, or ask a focused question if it materially affects the solution.
- Match depth to the assignment: a small exercise may only need to demonstrate a few of these concerns. Do not add architecture the task does not ask for.

---

## Details

### Product and company direction

Reindeer builds enterprise AI agents that execute business processes across existing systems. The emphasis is on real operational work, including exceptions, incomplete information, legacy systems, and human involvement.

A useful interpretation is a "system of work" operating across existing systems of record: gathering information, interpreting a case, coordinating actions, and moving a business process toward completion.

The broader challenge is making this capability reusable across organizations while accommodating each customer's systems, terminology, policies, permissions, and operational practices.

### Information shared in the candidate's interview

- Workflows may run for up to a week and involve humans along the way.
- When an agent lacks sufficient information, it can take additional steps and use tools to gather evidence before deciding.
- Human involvement is part of the process; the agent may need further information or judgment to continue.

These suggest that durable execution, context gathering, and pause-and-resume behavior are relevant considerations.

### Themes from previously reviewed company materials

Qualitative themes from prior research, not a verified description of the internal architecture:

- Enterprise processes contain exceptions and undocumented knowledge that simple happy-path automation misses.
- Relevant information is spread across business systems, documents, communications, and human experts.
- An agent may gather additional information, ask an expert a focused question with supporting context, and reassess the case after receiving an answer.
- Recurring exceptions can reveal opportunities to improve policies or processes. Proposed changes need evaluation and human approval.
- The execution environment around the model (the agent harness) needs to fit business workflows. Tool interfaces and context selection affect reliability, latency, and cost.
- Legacy integrations matter. Reviewed material discussed structured interaction with IBM i / AS/400 through 5250, illustrating the value of using available field and screen structure.

### Engineering implications (design inferences, not confirmed implementation)

**Customer-specific adaptation**: Separate reusable execution capabilities from customer-specific policies, integrations, field mappings, and process definitions. Consider how configurations evolve and what happens to in-flight work when a policy changes.

**Context quality and missing information**: An agent needs relevant, authorized, sufficiently current evidence. Structured queries, document retrieval, and human questions serve different needs. Missing information should remain explicit; a failed lookup should not become a business conclusion.

**Predictable execution and flexible decisions**: Known sequences can use explicit workflows. Model-driven decisions are useful where the next step depends on interpretation or newly discovered information. Business constraints and authorization should be enforced by application logic.

**Long-running execution and human involvement**: Consider durable state, correlation of external responses, waiting states, timeouts, and resumption after restarts. Human escalation should include a focused question, relevant evidence, and a clear description of what blocks progress.

**Reliable actions across systems**: Retries and partial failures can duplicate side effects. Consider idempotency, reconciliation, and the distinction between a failed action and an action whose outcome is unknown.

**Permissions and traceability**: Actions and retrieved information must respect the customer's access boundaries. Record relevant inputs, evidence, tool outcomes, decisions, policy versions, and human approvals so a case can be investigated.

**Evaluation and controlled improvement**: Changes to prompts, tools, retrieval, or policies can alter business outcomes. Evaluate representative cases and exceptions, including regression coverage, before expanding automation.

**Latency and cost**: Measure where time, tokens, and tool calls are spent. Potential improvements include preparing recurring context, narrowing retrieval, using supported prompt caching, and limiting unproductive loops. Evaluate efficiency improvements alongside outcome quality.
