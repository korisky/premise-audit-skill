# premise-audit

A Claude Code skill that stops the agent from answering the wrong question well.

Some requests aren't ready to be answered — not because they're badly written,
but because you don't yet know what you want. The usual failure isn't that the
agent gets it wrong; it's that it gets it *confidently right* about something
you didn't mean.

`premise-audit` makes the agent stop, read your actual code, and tell you what
you're assuming — before it writes anything.

## What it does

Invoked on an underspecified request, the agent produces exactly three things
and then stops:

1. **Unstated assumptions** you're making — ranked by blast radius, each one
   citing a real file and line
2. **Information that would change the answer** — each with a decision fork
   attached (if A then X, if B then Y)
3. **The common mistake** people make with this class of request

Then it asks one or two multiple-choice questions and **ends its turn**. No
implementation, no plan, no "anyway, here's what I'd do."

## Install

The skill directory name must be `premise-audit` to match the frontmatter, and
the repo is named `premise-audit-skill` — so pass the target path explicitly:

```bash
git clone https://github.com/korisky/premise-audit-skill.git \
  ~/.claude/skills/premise-audit
```

Project-local instead of global? Clone to `.claude/skills/premise-audit` in your
repo. Verify with `/premise-audit` — if it doesn't autocomplete, the directory
name is wrong.

## Usage

```
/premise-audit          # explicit
/premise-audit --quick  # assumptions only, no questions, no recon
```

It also fires on its own when a request is vague, high-stakes, or you say you're
not sure what you want. It's told to **abort** on mechanical, well-specified,
low-stakes work — auditing a typo fix is how a skill earns an uninstall.

Pairs well with plan mode: audit → answer → plan.

## What good output looks like

The whole point is grounding. Generic advice is a failure, not a partial
success:

> ❌ "You're assuming the current design scales."

> ✅ "You're assuming retries are safe to add at the transport layer — but
> `internal/client/post.go:88` sends a non-idempotent POST with no request key,
> so a retry on timeout double-charges. Nothing in `post_test.go` covers this."

If the agent could have written it without reading your code, it shouldn't have
written it.

## Design notes

**The stop is enforced, not requested.** `allowed-tools` omits `Edit` and
`Write`, so the agent *cannot* implement during an audit. Prose instructions to
hold back lose to action bias a meaningful fraction of the time; a missing tool
loses zero percent. The mandatory `AskUserQuestion` call is what actually ends
the turn.

**Multiple choice, not open-ended.** The premise is that you can't articulate
what you want — so an open question doesn't help; you'll answer it vaguely.
Recognition beats generation. Concrete options do the articulating for you.

**Recon is capped** at 3 searches and 5 files. Without a ceiling this becomes a
repo tour, and you've spent your context window on the preamble to the real
task.

## Why not call it "steelman"?

Steelmanning is constructing the strongest version of a position you *disagree
with*. This is a different move: excavating unstated premises before you commit
to any position. Naming it "steelman" pulls the behavior toward arguing the
other side, which is less useful here. The word appears in the skill description
so it still triggers if that's the phrase in your head.

## License

MIT
