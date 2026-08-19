---
name: premise-audit
description: Audit consequential ambiguity before answering or implementing. Identify assumptions, missing information that creates materially different paths, and one or two deciding questions. Use when explicitly requested, when the user is unsure what they want, or when a consequential request lacks a clear goal, success criterion, or constraint. Do not use for well-specified, mechanical, low-stakes, or easily reversible work.
---

# Premise Audit

Make the real question visible before answering it. The audit is the entire
response: do not answer, plan, or modify files.

## 1. Decide whether to audit

Always audit when the user explicitly invokes this skill or requests a premise
or assumption audit.

For automatic activation, audit only when missing information creates materially
different, consequential paths. Otherwise continue silently with the original
task.

## 2. Inspect bounded evidence

Use the host agent's available read and search capabilities. Open at most five
files and run at most three searches. Prioritize affected code, callers, tests,
configuration, and schemas.

For greenfield, tooling, or process requests, skip repository inspection. Use
stated conversational constraints and treat absent requirements as missing
evidence, not facts.

## 3. Write three sections

### Assumptions an answer would currently require

Rank 3-5 assumptions by blast radius, using fewer when fewer are justified. Each
item must:

- state what the agent would have to assume
- cite the evidence that makes it uncertain
- explain what changes or fails if it is false

Cite `path/file.go:88` when repository evidence exists. Otherwise cite the
relevant conversational constraint or identify the missing fact explicitly.

> To add retries at the transport layer, I would have to assume requests are
> idempotent. `internal/client/post.go:88` sends a POST without an idempotency key,
> so that assumption being false can duplicate a charge.

### Information that would significantly change the answer

List 2-4 missing facts. Give each a decision fork: if A, the recommendation moves
toward X; if B, it moves toward Y. Exclude information that would not change the
answer.

### Most likely failure mode here

Describe one specific failure mode tied to an assumption or decision fork above.
Do not claim prevalence without evidence.

## 4. Ask the deciding questions

Ask only the 1-2 questions whose answers change what would be built or
recommended.

- Give 3-4 concrete, distinct options per question.
- Put the recommended option first and mark it `(Recommended)`.
- Omit `Other` when the host supplies it automatically.
- Use a structured question tool when available; otherwise print the options in
  prose after the three sections.

> What should happen when a retry meets an in-flight request?
>
> 1. Use a client idempotency key and deduplicate server-side (Recommended)
> 2. Fail closed and let the caller decide whether to retry
> 3. Accept duplicates and reconcile them later

## 5. Stop

End after the questions. Do not answer the original request or append a plan. If
a structured question tool returns an answer in the same execution, do not resume
the original task until the user's next message.

After the user answers, handle the clarified request normally. If one
consequential fork remains, ask one narrower follow-up instead of repeating the
audit.

## Quick mode

When invoked with `--quick`, or when the user requests assumptions without
questions, emit only the assumptions section. Use at most one targeted lookup,
ask no question, and do not answer the original request.
