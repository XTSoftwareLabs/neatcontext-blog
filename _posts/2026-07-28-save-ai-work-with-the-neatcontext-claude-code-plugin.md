---
layout: post
title: "Stop Losing Your Best AI Work: Introducing the NeatContext Plugin for Claude Code"
date: 2026-07-28
categories: product
---

You have had this conversation before. You spend an hour with Claude Code working through a gnarly production issue — correlating traces, ruling out red herrings, finally landing on the real root cause. The fix ships. The chat window closes. Three weeks later a similar issue shows up, on-call is someone else this time, and the investigation starts over from zero. All of that hard-won reasoning is gone, or worse, buried in a chat history nobody will think to search.

This is the gap [neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins) closes. It is a Claude Code plugin — [NeatContext for Claude Code](https://github.com/XTSoftwareLabs/neatcontext-plugins) — built to solve one specific problem: **the useful work and domain knowledge produced inside an AI conversation disappears the moment the conversation ends.**

## The problem: good conversations are a dead end

Working through a hard problem with an AI client generates real knowledge — not just the final answer, but the path there. Which signals mattered together. Which assumptions turned out to be wrong. Which systems you had to check and in what order. That is exactly the kind of domain knowledge that makes a future answer accurate instead of generic.

The trouble is where that knowledge ends up living:

- It's trapped in one chat session, on one person's machine.
- Resuming the conversation gets you the history back, but not a distilled, reusable form of it — you are re-reading the whole back-and-forth to find the part that mattered.
- Exporting a transcript preserves the raw record, but a transcript is not something Claude can efficiently reason from next time. It's noise wrapped around a small amount of signal.

So teams either re-explain the same context every time, or they lose it entirely when the person who had it moves on to the next ticket. Either way, the next engineer — or the same engineer three weeks later — starts from scratch.

## What NeatContext does about it

NeatContext extracts the durable knowledge from a Claude Code conversation and saves it as a structured, reusable context: a domain profile capturing the rules and conventions that shaped how Claude behaved, plus a knowledge folder with the TSGs, runbooks, findings, and notes worth keeping. Not the whole transcript — the part that will actually help next time.

That distinction matters, so it's worth being explicit about it:

| | Best for | What you get |
|---|---|---|
| **NeatContext** | Reusing knowledge in fresh sessions or across a team | A generated domain profile plus a knowledge folder — reusable context, not a transcript |
| **Claude Code resume** | Continuing the same conversation | The original session and its full conversation history |
| **Save or export a conversation** | Keeping a record | The raw transcript, including all the back-and-forth that led to the result |

Once a context is saved, you connect it in a new session — or hand it to a teammate — and Claude starts already knowing what your last long, expensive conversation figured out.

## What it looks like in practice

Say you just finished tracking down a nasty hot-partition bug: order updates were lagging on one partition while the aggregate metric looked fine, and it took correlating traces, per-partition lag, and worker deserialization logs to find it. Instead of letting that reasoning evaporate when you close the tab:

```text
You: /neatcontext:save event-partition-investigation

Claude:
Lite context folder: <folder>
Profile path: <folder>/profile.md
Knowledge folder: <folder>/knowledge
Use command: /neatcontext:use event-partition-investigation
```

Weeks later, a different but related symptom shows up — shipment updates delayed, queue lag looking normal. Connect the saved context and Claude starts from where the last investigation left off, instead of guessing:

```text
You: /neatcontext:use event-partition-investigation

Claude: Connected to event-partition-investigation.

You: Shipment updates are delayed, but overall queue lag is low. Help me investigate.

Claude: I will start with the checks from the saved context: per-partition lag,
        event size, partition keys, and deserialization time.
```

That's the whole idea: the second investigation inherits the first one's hard-earned knowledge, without anyone having to re-explain it or dig through old chat logs.

## The commands

The plugin is a small set of slash commands, all scoped to the current Claude Code session:

- **`/neatcontext:save [name]`** — save the useful work in the current conversation as a new context.
- **`/neatcontext:use [name or number]`** — connect a saved context to the current session.
- **`/neatcontext:list`** — list every context you can connect to.
- **`/neatcontext:status`** — show what's connected and flag any problems with it.
- **`/neatcontext:create`** — build a fresh context from an existing knowledge folder, without needing a conversation to extract from.
- **`/neatcontext:import [folder]`** — bring in a context a teammate shared with you.
- **`/neatcontext:delete [name or number]`** — remove a context you no longer need.
- **`/neatcontext:mode [auto|ask|manual]`** — control whether Claude switches contexts automatically, asks first, or waits for you to say so.

## Install it

```bash
claude plugin marketplace add https://github.com/XTSoftwareLabs/neatcontext-plugins.git
claude plugin install neatcontext@neatcontext --scope user
```

Then restart Claude Code (or run the equivalent `/plugin marketplace add` and `/plugin install` commands from inside a session). It needs Claude Code 2.1.196+ and Node.js 18+; the NeatContext desktop app is only required if you also want to connect the richer standard contexts it manages.

## Where knowledge is stored, and what stays local

Everything a lite context needs — the profile and knowledge folder — is stored locally under `~/.neatcontext/lite`. The plugin runs only the Node.js files bundled in the repository and makes no outbound internet requests itself; its desktop integration talks to the NeatContext companion app over `127.0.0.1`. Full details are in the project's [privacy policy](https://github.com/XTSoftwareLabs/neatcontext-plugins/blob/main/PRIVACY.md).

## Why this matters beyond one engineer

This plugin is the individual-session counterpart to the broader problem we've written about before: [why more context is not always better context](https://blog.neatcontext.com/ai/2026/07/14/why-more-context-is-not-always-better-context/) and [how to build efficient context for your AI client](https://blog.neatcontext.com/guide/2026/07/22/how-to-build-efficient-context-for-ai-clients/). A good context needs rules, knowledge, and retrieval — and one of the most reliable sources of real, team-specific knowledge is the work your team has *already done* with an AI client. NeatContext makes that knowledge durable instead of throwaway, and shareable instead of stuck in one person's history.

## Try it

The repo is open source and MIT-licensed: [github.com/XTSoftwareLabs/neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins). Install it, run `/neatcontext:save` after your next real debugging session, and see what it looks like to have Claude start your next one already knowing what the last one learned. We would love your feedback.
