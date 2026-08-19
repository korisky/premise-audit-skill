---
name: premise-audit
description: Surfaces the unstated assumptions, missing information, and typical failure mode behind an underspecified request, then asks the one or two questions that would actually change the answer — before writing any code. Also known as the steelman check or premise check. Use when the user says they are not sure what they really want, when a request is vague, open-ended, or architecturally high-stakes ("should we", "what's the best way to", "design X", "how do I approach"), or when the user invokes /premise-audit. Do NOT use for well-specified, mechanical, or low-stakes tasks.
allowed-tools: Read, Grep, Glob, Bash, AskUserQuestion, mcp__semble__search, mcp__semble__find_related, mcp__codegraph__codegraph_explore
---

# Premise Audit

The user has asked for something before they are sure what they want. Your job
is **not** to answer. Your job is to make the real question visible, then stop.

The output is only worth reading if it is grounded in **this** codebase. Generic
consulting-speak ("you're assuming your users want this") is a failure, not a
partial success. Every claim cites a file, a line, or an observed fact.

## Step 0 — Abort check

Before anything else, decide whether this request actually needs an audit.

**Abort and just do the work** if the request is:
- mechanically specified (rename this, fix this typo, add this field)
- low-stakes and easily reversed
- already accompanied by clear constraints and acceptance criteria

If aborting, say so in one line — "This is well-specified; skipping the premise
audit and proceeding." — and get on with the task. Do not audit a two-line fix.
A skill that fires on everything gets uninstalled.

## Step 1 — Bounded recon

Read enough of the real system to say something specific. Budget, and hold it:

- ≤ 3 searches
- ≤ 5 files actually opened
- ≤ 2 minutes of shell (dependency manifest, test layout, git log on the
  relevant paths, whatever is cheap and load-bearing)

Prioritise: the code the request would touch, its callers, its tests, and any
config or schema that pins behaviour. Skip anything you will not cite.

If the request is not about an existing codebase (greenfield, tooling choice,
process question), skip recon and ground the audit in the constraints the user
has already stated elsewhere in the conversation instead.

## Step 2 — Emit exactly three sections

### 1. Assumptions you are making that you haven't stated

Rank by **blast radius** — what breaks worst if the assumption is wrong — not by
how clever the observation is. Aim for 3–5. Each one:

- names the assumption in the user's own terms
- cites the concrete thing that makes it questionable (`path/file.go:88`, a
  dependency version, a missing test, a schema constraint)
- states the consequence if it turns out false

> Bad: "You're assuming the current design scales."
> Good: "You're assuming retries are safe to add at the transport layer — but
> `internal/client/post.go:88` sends a non-idempotent POST with no request key,
> so a retry on timeout double-charges. Nothing in `post_test.go` covers this."

### 2. Information that would significantly change the answer

Not a wishlist. Each item must have a **decision fork** attached: if it's A the
answer is X, if it's B the answer is Y. If you cannot name both branches, the
information does not belong on this list. Aim for 2–4.

### 3. The most common mistake with this class of request

One paragraph. The failure mode people walk into when they ask *this kind* of
question — the local-maximum solution, the premature abstraction, the thing that
works until the second use case. Be specific about the class; "not thinking it
through" is not an answer.

## Step 3 — Ask, with options

Call `AskUserQuestion`. This is mandatory and it is the mechanism that ends your
turn — do not replace it with a question in prose.

Rules:
- **1–2 questions maximum.** Only the ones whose answers change what you'd build.
- 3–4 options each, and each option must be a genuinely different path, not a
  shade of the same one.
- Options must be **concrete and recognisable**, because the user's stated
  problem is that they can't articulate what they want. Recognition is easier
  than generation — do the articulating for them.
- Lead with your recommendation and mark it `(Recommended)`.
- Do not add an "Other" option; the harness supplies one.

> Weak: "What are your priorities here?"
> Strong: "What should happen when a retry hits an in-flight request?"
> — Idempotency key on the client, dedupe server-side (Recommended)
> — Fail closed: surface the timeout, let the caller decide
> — Accept the duplicate; reconcile in the nightly job

## Step 4 — Hard stop

After `AskUserQuestion`, **end your turn**. Do not answer the original question.
Do not draft the implementation "to save a round trip." Do not append a plan.
The audit is the entire deliverable for this turn.

Once the user answers, the audit is complete and normal work resumes — proceed
with the actual task, informed by what you just learned. Pairs well with plan
mode: audit → answer → plan.

## Quick mode

If invoked as `/premise-audit --quick`, or if the user asks for the assumptions
without the interrogation: emit section 1 only, skip recon beyond a single
search, skip `AskUserQuestion`, and stop. No question, no answer.

## Anti-patterns

- **Fortune cookie.** Assumptions that would apply to any project. If you could
  have written it without reading the code, delete it.
- **Padding to five.** Three sharp assumptions beat five with two invented ones.
- **Interrogation.** More than two questions and the user gives up and says
  "just do whatever." One good question outperforms four mediocre ones.
- **Sneaking in the answer.** "You're assuming X — anyway, here's the
  implementation." No. Stop at the question.
- **Recon sprawl.** Reading thirty files to audit a request is its own failure;
  you have spent the user's context on the preamble.
