---
name: premise-audit
description: Audit consequential ambiguity before answering or implementing. Surface the assumptions an answer would require, identify missing information that creates materially different paths, and ask one or two concrete choice questions. Use when the user explicitly requests a premise or assumption audit, says they are unsure what they want, or leaves an unresolved goal, success criterion, or high-cost architectural decision. Do not auto-activate merely because a request says "should we" or "how should I"; skip automatic use for well-specified, mechanical, low-stakes, or easily reversible work.
---

# Premise Audit

Make the real question visible before answering it. Do not answer the original
question, propose a plan, or modify files during the audit.

Ground the audit in available evidence. For an existing codebase, cite files,
lines, tests, configuration, schemas, dependencies, or observed behavior. For a
greenfield or process request, cite stated conversational constraints and label
important missing evidence explicitly. Generic consulting advice is a failure.

## Step 0: Decide whether to audit

Always audit when the user explicitly invokes this skill or asks for a premise
or assumption audit.

For automatic activation, continue with the original task without an audit when
it is:

- mechanically specified
- low-stakes and easily reversed
- accompanied by clear constraints and acceptance criteria
- missing no information that would materially change the answer

Do not announce that an automatically activated audit was skipped. A skill that
interrupts clear work will be disabled.

## Step 1: Run bounded reconnaissance

Read only enough of the system to identify consequential decision forks. Use
whatever read and search capabilities the host agent provides. Keep within:

- 3 searches
- 5 opened files
- 2 minutes of shell exploration

Prioritize the code the request would affect, its callers, its tests, and any
configuration or schema that constrains behavior. Do not inspect material that
will not support the audit.

For greenfield, tooling, or process questions, skip repository reconnaissance.
Use facts and constraints already present in the conversation. Treat absent
requirements as missing evidence, not as facts.

## Step 2: Write the audit

Emit these three sections in order.

### 1. Assumptions an answer would currently require

Rank 3-5 assumptions by blast radius. Use fewer when fewer are justified. Each
item must:

- state what the agent would have to assume, in the user's terms
- cite the evidence that makes the assumption uncertain
- state what fails or changes if the assumption is false

Use `path/file.go:88` citations when repository evidence exists. For non-code
requests, identify the relevant stated constraint or explicitly missing fact.

> Weak: "You are assuming the current design scales."
>
> Strong: "To add retries at the transport layer, I would have to assume requests
> are idempotent. `internal/client/post.go:88` sends a POST without an idempotency
> key, so that assumption being false can duplicate a charge."

### 2. Information that would significantly change the answer

List 2-4 missing facts. Attach a decision fork to each: if the answer is A, the
recommendation moves toward X; if it is B, it moves toward Y. Exclude anything
that would not alter the recommendation or implementation.

### 3. Most likely failure mode here

Write one paragraph describing the specific failure mode this request is at risk
of. Tie it to an assumption or decision fork already identified. Do not claim a
failure is common or statistically likely without evidence.

## Step 3: Ask the deciding questions

Ask no more than 1-2 questions. Ask only questions whose answers change what
would be built or recommended.

- Provide 3-4 concrete, genuinely different options for each question.
- Put the recommended option first and mark it `(Recommended)`.
- Prefer recognition over generation: articulate the viable paths for the user.
- Do not add an `Other` option when the host supplies one automatically.
- Use a structured question tool when one is available. Otherwise, render the
  questions and options in prose after the three audit sections.

> Weak: "What are your priorities here?"
>
> Strong: "What should happen when a retry meets an in-flight request?"
>
> 1. Use a client idempotency key and deduplicate server-side (Recommended)
> 2. Fail closed and let the caller decide whether to retry
> 3. Accept duplicates and reconcile them later

## Step 4: Stop

After presenting the questions, end the response. Do not answer the original
request, draft an implementation, or append a plan. If a structured question
tool returns an answer during the same execution, retain the answer but still do
not proceed with the original task until the user's next message.

After the user answers, respond to the clarified request normally. Do not assume
they want implementation unless they asked for it. If their answer exposes one
new consequential fork, ask one narrower follow-up instead of repeating the full
audit.

## Quick mode

When invoked with `--quick`, or when the user requests assumptions without
questions, emit section 1 only. Use at most one targeted lookup, ask no question,
and do not answer the original request.

## Anti-patterns

- **Fortune cookie:** Delete assumptions that could have been written without
  reading the available evidence.
- **Padding:** Three supported assumptions beat five with two invented ones.
- **Interrogation:** More than two questions pushes the work back onto the user.
- **Sneaking in the answer:** The audit is the deliverable for this response.
- **Reconnaissance sprawl:** Do not spend the context budget touring the repo.
