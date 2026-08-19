# premise-audit

A portable Agent Skill that stops a code agent from giving a good answer to the
wrong question.

It identifies assumptions the agent would otherwise make, shows which missing
facts would change the answer, and asks one or two concrete questions before any
implementation begins.

## What it does

For an underspecified request, the agent produces:

1. assumptions an answer would require, grounded in repository or conversational
   evidence
2. missing information with explicit decision forks
3. the most likely failure mode tied to those assumptions

It then asks one or two multiple-choice questions and stops. It does not answer,
plan, or modify files during the audit.

## Install

Clone this repository to one of the following paths. The destination directory
must be named `premise-audit`.

| Agent | Personal | Project |
| --- | --- | --- |
| Codex | `~/.codex/skills/premise-audit` | `.agents/skills/premise-audit` |
| Claude Code | `~/.claude/skills/premise-audit` | `.claude/skills/premise-audit` |
| [OpenCode](https://opencode.ai/docs/skills) | `~/.config/opencode/skills/premise-audit` | `.opencode/skills/premise-audit` |
| [Pi](https://pi.dev/docs/latest/skills) | `~/.pi/agent/skills/premise-audit` | `.pi/skills/premise-audit` |

For example:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/korisky/premise-audit-skill.git \
  ~/.codex/skills/premise-audit
```

## Usage

| Agent | Full audit | Quick mode |
| --- | --- | --- |
| Claude Code, OpenCode | `/premise-audit` | `/premise-audit --quick` |
| Pi | `/skill:premise-audit` | `/skill:premise-audit --quick` |
| Codex | `Use premise-audit before answering.` | `Use premise-audit in quick mode.` |

Agents may also activate the skill automatically when a consequential request
contains unresolved decisions. Mechanical, well-specified, low-stakes, and
easily reversible tasks proceed normally.

## Example

> To add retries at the transport layer, I would have to assume requests are
> idempotent. `internal/client/post.go:88` sends a POST without an idempotency key,
> so that assumption being false can duplicate a charge.

The assumption belongs to the answer the agent might give, not necessarily to
the user. For greenfield requests, the agent uses stated constraints and missing
facts instead of inventing repository evidence.

## Portability

The skill uses no agent-specific tools. Structured questions fall back to prose.
Its stop rule is behavioral; clients that require a technical write lock must
enforce one through their own permissions or hooks.
