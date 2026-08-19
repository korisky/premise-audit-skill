# premise-audit

An Agent Skills-compatible workflow for stopping a code agent from answering the
wrong question well.

People often bring an agent a request before the goal, constraints, or acceptable
tradeoffs are fully clear. The usual failure is not a nonsensical answer. It is a
confident, technically sound answer to a question nobody meant to ask.

`premise-audit` makes the agent inspect the available evidence, expose the
assumptions it would otherwise make, and ask only the questions that lead to
materially different decisions.

## What it does

For an underspecified request, the agent produces:

1. **Assumptions an answer would require**, ranked by blast radius and grounded
   in repository or conversational evidence
2. **Missing information that changes the answer**, with the decision fork made
   explicit
3. **The most likely failure mode here**, tied to the identified assumptions

It then asks one or two concrete choice questions and stops. It does not answer
the original request, propose a plan, or modify files during the audit.

For an existing codebase, findings cite files, lines, tests, configuration, or
observed behavior. For greenfield and process questions, findings cite stated
constraints and identify missing evidence explicitly.

## Install

This repository follows the open [Agent Skills specification](https://agentskills.io/specification).
Clone it into a skills directory recognized by your agent. The destination
directory must be named `premise-audit`.

### Codex

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/korisky/premise-audit-skill.git \
  ~/.codex/skills/premise-audit
```

For a project-local installation, use `.agents/skills/premise-audit` when your
Codex environment discovers project skills from that location.

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/korisky/premise-audit-skill.git \
  ~/.claude/skills/premise-audit
```

For a project-local installation, use `.claude/skills/premise-audit`.

### GitHub Copilot and VS Code

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/korisky/premise-audit-skill.git \
  ~/.agents/skills/premise-audit
```

GitHub Copilot also supports `~/.copilot/skills/premise-audit` for personal use
and `.github/skills/premise-audit` or `.agents/skills/premise-audit` in a
project. See GitHub's [agent skills documentation](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills).

Other Agent Skills clients may use different discovery paths. Point the client
at this directory rather than copying only `SKILL.md`, so future bundled
resources remain available.

## Usage

Agents may activate the skill automatically when a request contains a
consequential unresolved decision. On clients that expose skills as commands:

```text
/premise-audit
/premise-audit --quick
```

You can also ask the agent directly:

```text
Use premise-audit before answering this request.
```

Quick mode returns only the assumptions section, performs at most one targeted
lookup, asks no questions, and stops.

Automatic activation is deliberately conservative. Phrases such as "should we"
or "how should I" are not sufficient by themselves. Mechanical, well-specified,
low-stakes, and easily reversible tasks should proceed without interruption.

## What good output looks like

The key is evidence and attribution. The skill identifies assumptions the agent
would need to make; it does not accuse the user of secretly holding them.

> Weak: "You are assuming the current design scales."

> Strong: "To add retries at the transport layer, I would have to assume requests
> are idempotent. `internal/client/post.go:88` sends a POST without an idempotency
> key, so that assumption being false can duplicate a charge."

If the agent could have written the finding without inspecting the available
evidence, it should not have written it.

## Portability and enforcement

The portable skill intentionally does not declare `allowed-tools`. Tool names,
permission semantics, and structured-question APIs differ among agents, and the
standard field is currently experimental.

The stop is a behavioral instruction. A `SKILL.md` file alone cannot guarantee a
write lock or turn boundary across clients. Environments that require technical
enforcement should add client-specific permissions or hooks that deny writes
while the audit is active.

When a host offers a structured multiple-choice question tool, the skill uses
it. Otherwise it prints the same options in prose and ends the response.

## Why not call it "steelman"?

Steelmanning constructs the strongest version of a position one disagrees with.
This workflow instead uncovers premises before committing to a position. The
description still uses familiar assumption-audit language for discovery without
pulling the behavior toward debating the user.
