---
layout: post
title: "Trust Your Team's Domain Knowledge, Not an AI Memory Database"
date: 2026-08-06
categories: product
---

A lot of AI tools now ship "memory": the assistant quietly writes facts about you, your project, or your team into a database, then pulls them back into later conversations. It sounds like the fix for the thing everyone complains about — starting from zero every session. It has a flaw that gets worse the longer you use it: nobody reviewed what got written down.

Say an engineer tells Claude, mid-incident, "this is probably the connection pool again." Memory files that away as a fact. Three weeks later, a different engineer asks about a slow endpoint on a different service, and the assistant opens with the connection pool theory — not because it's true this time, but because memory says it's true, generally. Nobody approved that. It just accumulated, the way a rumor accumulates, and now it shapes every answer that follows.

## The problem isn't only accuracy. It's that nothing gets checked.

Every source of information is wrong sometimes — runbooks, wikis, people. That's not unusual. What's unusual about a memory database is that there's no step where a human looks at a fact before the AI starts treating it as ground truth. The same model that might get an answer wrong is also the one deciding what's worth remembering, writing it to a store you don't naturally read, and retrieving it later with full confidence. Errors don't get caught. They compound, quietly, inside a place you can't easily open up and diff.

This isn't specific to one vendor. Whether it's a built-in assistant memory or a third-party memory service that persists to its own database, the shape is the same: extraction is automatic, storage is a row in someone else's DB, and there's no review gate in between. A better extraction model doesn't fix that. The mechanism is missing a trust step, not a smarter model.

## Your team already has a mechanism for this

You don't trust a line of code because a machine wrote it and stored it somewhere. You trust it because it's a file you can read, a diff someone else looked at, and a pull request that got approved before it landed on `main`. That's not a new process — it's the one every engineering team already runs, all day, for exactly this reason: nothing becomes "true for the team" until a person says so.

Domain knowledge deserves the same bar as the code it describes. That's the idea behind NeatContext.

## How NeatContext does it

NeatContext doesn't put your team's knowledge in a database. `/neatcontext:save` writes it out as plain Markdown — a `profile.md` for the rules, a `knowledge/` folder for the findings — files you can open, read, and diff like anything else in your repo. Because it's just files, it goes through the git workflow your team already trusts:

```text
 AI conversation
        │  /neatcontext:save <name>
        ▼
 Local context (profile.md + knowledge/*.md)
        │  /neatcontext:export <name> --to contexts/
        ▼
 contexts/<name>/  in your repo
        │  git commit, push, open PR
        ▼
 Pull request ──review──▶ merge
        │
        ▼
 Team repo (main)
        │  git pull → /neatcontext:import → /neatcontext:use
        ▼
 Teammate's session ── asks AI, gets a reviewed answer
        │
        │  works for a while, learns more
        │  /neatcontext:save <name>        (updates it)
        │  /neatcontext:export <name> --to contexts/ --force
        ▼
 back to "Pull request" ↺  (a new diff, reviewed again)
```

Here's what that looks like step by step.

**1. Have the conversation, then save what you learned.**

You work an investigation with Claude the normal way — checking logs, ruling out guesses, finding the actual cause. When you're done, you save it:

```text
You: /neatcontext:save checkout-pool-exhaustion

Claude:
Context folder: ~/.neatcontext/contexts/checkout-pool-exhaustion-a1b2c3
Profile path: ~/.neatcontext/contexts/checkout-pool-exhaustion-a1b2c3/profile.md
Knowledge folder: ~/.neatcontext/contexts/checkout-pool-exhaustion-a1b2c3/knowledge
Use command: /neatcontext:use checkout-pool-exhaustion
```

That lives on your machine only — nobody else can see it yet.

**2. Export it as a bundle you can hand off.**

```text
You: /neatcontext:export checkout-pool-exhaustion --to contexts

Claude: Exported "checkout-pool-exhaustion" to contexts/checkout-pool-exhaustion
```

This copies the profile and knowledge into a self-contained folder inside your repo — the shareable form of what you just saved.

**3. Put it in git and open a PR. A teammate reviews the actual knowledge, not a black box.**

```bash
git checkout -b add-checkout-pool-exhaustion-context
git add contexts/checkout-pool-exhaustion
git commit -m "Add checkout pool exhaustion context"
git push -u origin add-checkout-pool-exhaustion-context
```

A reviewer reads `knowledge/pool-exhaustion.md` the way they'd read a runbook change, and leaves the kind of comment a wiki edit never gets: *"the default_pool_size here is stale — infra bumped it last week, update the number before this merges."* You fix it, push again, they approve, it merges. Nothing about pool exhaustion is "team truth" until that happens.

**4. A teammate on a different task pulls it in and connects it.**

```text
$ git pull

You: /neatcontext:import --from contexts/checkout-pool-exhaustion
Claude: Imported "checkout-pool-exhaustion".

You: /neatcontext:use checkout-pool-exhaustion
Claude: Connected to checkout-pool-exhaustion.

You: Checkout is timing out, same symptoms as last time. What should I check first?

Claude: Check pgbouncer's default_pool_size on billing-postgres first — this
        symptom traced back to pool exhaustion before, and the reviewed note
        has the current pool size, not the stale one.
```

The answer is accurate not because the model got smarter — it's the same model as always. It's accurate because the fact it's drawing on already passed a human review.

**5. They work with it for a while, then save their own findings back.**

```text
You: /neatcontext:save checkout-pool-exhaustion

Claude: This will update "checkout-pool-exhaustion":
  - knowledge/pool-exhaustion.md: adds the monitoring alert added after this fix
Confirm? (y/n)

You: y
Claude: Updated "checkout-pool-exhaustion".

You: /neatcontext:export checkout-pool-exhaustion --to contexts --force
Claude: Exported "checkout-pool-exhaustion" to contexts/checkout-pool-exhaustion (overwritten).
```

**6. That update goes through review too — the cycle just repeats.**

```bash
git checkout -b update-checkout-pool-exhaustion-context
git add contexts/checkout-pool-exhaustion
git commit -m "Update checkout pool exhaustion context: add monitoring alert"
git push -u origin update-checkout-pool-exhaustion-context
```

Same PR, same review, same merge. Whoever pulls the repo next — a third engineer, or the same one in three months — inherits the update automatically. `git blame` shows who wrote each part and who approved it. Nothing decays into an unreviewed blob; it stays a sequence of reviewed diffs, same as the code it sits next to.

## Why this actually holds up

| | Memory database | NeatContext (git-reviewed) |
|---|---|---|
| Who approves a fact before it's trusted | No one — the model writes and reads its own extraction | A named reviewer, in a PR |
| How you check what's stored | Opaque, vendor-specific storage | Plain Markdown files, open and read them directly |
| How you fix something wrong | Unclear, often no direct edit path | A diff, a PR, a merge — same as fixing code |
| How it reaches the team | Tied to one account or one chat | Checked in, shared, importable by anyone with repo access |

None of this is exotic. It's the same trust model your team already applies to the code it ships. The only thing that changes is applying it to the knowledge an AI reasons from, instead of leaving that knowledge to accumulate, unreviewed, in a database.

Give it a try: the plugin is open source at [github.com/XTSoftwareLabs/neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins), and the desktop app's [team library](https://docs.neatcontext.com/features/library#the-team-library-optional-read-only) covers the same idea for context beyond a single Claude Code chat. We would love your feedback.
