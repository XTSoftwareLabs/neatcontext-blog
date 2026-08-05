---
layout: post
title: "Trust Your Team's Domain Knowledge, Not an AI Memory Database"
date: 2026-08-06
categories: product
---

A lot of AI tools now ship "memory": the assistant quietly writes facts about you, your project, or your team into a database, then pulls them back into later conversations. It sounds like the fix for the thing everyone complains about — starting from zero every session. It has a flaw that gets worse the longer you use it: nobody reviewed what got written down.

Say an engineer tells Claude, mid-incident, "this is probably the connection pool again." Memory files that away as a fact. Three weeks later, a different engineer asks about a slow endpoint on a different service, and the assistant opens with the connection pool theory — not because it's true this time, but because memory says it's true, generally. Nobody approved that. It just accumulated, the way a rumor accumulates, and now it shapes every answer that follows.

## The problem isn't only accuracy. It's that no one can check it.

Every source of information is wrong sometimes — runbooks, wikis, people. That's not unusual. What's unusual about a memory database is that there's no step where a human looks at a fact before the AI starts treating it as ground truth. The same model that might get an answer wrong is also the one deciding what's worth remembering, writing it to a store you don't naturally read, and retrieving it later with full confidence. Errors don't get caught. They compound, quietly, inside a place you can't easily open up and diff.

This isn't specific to one vendor. Whether it's a built-in assistant memory or a third-party memory service that persists to its own database, the shape is the same: extraction is automatic, storage is a row in someone else's DB, and there's no review gate in between. A better extraction model doesn't fix that. The mechanism is missing a trust step, not a smarter model.

## Your team already has a mechanism for this

You don't trust a line of code because a machine wrote it and stored it somewhere. You trust it because it's a file you can read, a diff someone else looked at, and a pull request that got approved before it landed on `main`. That's not a new process — it's the one every engineering team already runs, all day, for exactly this reason: nothing becomes "true for the team" until a person says so.

Domain knowledge deserves the same bar as the code it describes. That's the idea behind NeatContext.

## How NeatContext does it

NeatContext doesn't put your team's knowledge in a database. `/neatcontext:save` writes it out as plain Markdown — a `profile.md` for the rules, a `knowledge/` folder for the findings — files you can open, read, and diff like anything else in your repo. Because it's just files, it goes through the git workflow your team already trusts.

**1. Save what you learned from a real investigation.**

```text
You: /neatcontext:save checkout-pool-exhaustion

Claude:
Lite context folder: contexts/checkout-pool-exhaustion
Profile path: contexts/checkout-pool-exhaustion/profile.md
Knowledge folder: contexts/checkout-pool-exhaustion/knowledge
```

**2. Put it in git, like any other change.**

```bash
git checkout -b add-checkout-pool-exhaustion-context
git add contexts/checkout-pool-exhaustion
git commit -m "Add checkout pool exhaustion context"
git push -u origin add-checkout-pool-exhaustion-context
```

**3. Open a PR. A teammate reviews the actual knowledge, not a black box.**

They read `knowledge/pool-exhaustion.md` the way they'd read a runbook change, and leave the kind of comment a wiki edit never gets: *"the default_pool_size here is stale — infra bumped it last week, update the number before this merges."* You fix it, push again, they approve, it merges. Nothing about pool exhaustion is "team truth" until that happens.

**4. A teammate picks up the reviewed knowledge.**

```text
$ git pull

You: /neatcontext:import contexts/checkout-pool-exhaustion
Claude: Imported checkout-pool-exhaustion.

You: /neatcontext:use checkout-pool-exhaustion
Claude: Connected to checkout-pool-exhaustion.
```

The next engineer on call starts from knowledge that passed review — not from a memory blob nobody signed off on.

**5. When it goes stale, fix it the same way.** Another PR, another diff against `knowledge/pool-exhaustion.md`, another approval. `git blame` shows who wrote it, who approved it, and when. Anyone can revert it if it turns out to be wrong.

## Why this actually holds up

| | Memory database | NeatContext (git-reviewed) |
|---|---|---|
| Who approves a fact before it's trusted | No one — the model writes and reads its own extraction | A named reviewer, in a PR |
| How you check what's stored | Opaque, vendor-specific storage | Plain Markdown files, open and read them directly |
| How you fix something wrong | Unclear, often no direct edit path | A diff, a PR, a merge — same as fixing code |
| How it reaches the team | Tied to one account or one chat | Checked in, shared, importable by anyone with repo access |

None of this is exotic. It's the same trust model your team already applies to the code it ships. The only thing that changes is applying it to the knowledge an AI reasons from, instead of leaving that knowledge to accumulate, unreviewed, in a database.

Give it a try: the plugin is open source at [github.com/XTSoftwareLabs/neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins), and the desktop app's [team library](https://docs.neatcontext.com/features/library#the-team-library-optional-read-only) covers the same idea for context beyond a single Claude Code chat. We would love your feedback.
