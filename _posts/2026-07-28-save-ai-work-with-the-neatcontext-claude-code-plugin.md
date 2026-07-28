---
layout: post
title: "Stop Losing Your Best AI Work: Introducing the NeatContext Plugin for Claude Code"
date: 2026-07-28
categories: product
---

You have had this moment before. You spend an hour with Claude Code working through a hard production issue. You check traces, rule out wrong guesses, and finally find the real cause. You fix it. You close the chat. Three weeks later, a similar issue shows up. It's someone else's turn to be on call this time, and they start from zero. All the hard work from before is gone — or it's buried in a chat history that nobody will think to search.

This is the problem [neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins) is built to fix. It is a Claude Code plugin — [NeatContext for Claude Code](https://github.com/XTSoftwareLabs/neatcontext-plugins) — and it solves one specific problem: **the useful work and knowledge you build up during an AI conversation gets lost once the conversation ends.**

## The problem: good work gets lost

When you work through a hard problem with an AI, you learn things along the way — not just the final answer, but how you got there. Which clues mattered. Which guesses turned out wrong. Which systems you had to check, and in what order. This is exactly the kind of knowledge that helps you get a good answer faster next time.

The problem is where that knowledge ends up:

- It stays inside one chat, on one person's computer.
- Resuming the chat gives you back the full history, but not a short, useful summary of it. You still have to read through everything to find the part that matters.
- Saving or exporting the chat keeps a full copy, but a full chat log is not easy for Claude to use later. It's mostly noise with a small amount of useful signal buried inside.

So teams end up doing one of two things: explaining the same background over and over, or losing it completely once the person who knows it moves to a different task. Either way, the next person — or even the same person, weeks later — starts over.

## More than a handoff, and more than compacting

A lot of effort right now goes into a related problem: keeping one long conversation working well. As a chat grows, it fills up the AI's context window. Tools deal with this by compacting — quietly summarizing older messages to make room for new ones — or by handoff, packing up a conversation so a new session (or a teammate) can pick up where you left off. Both are useful, and both are solving a real pain point: a chat that has grown too long to keep working with as-is.

NeatContext's saved context happens to solve that same pain point, just as a side effect of how it works. Because `/neatcontext:save` pulls out only the durable knowledge — the rules and the findings — and leaves the rest of the back-and-forth behind, what you get is already small and focused enough to hand to a fresh session without hitting context-window limits. Connect it with `/neatcontext:use` and Claude picks up right where the task left off. So it covers the handoff case, and it sidesteps the "conversation got too long" problem too, without needing a separate compacting step: you never carry the bloat forward in the first place, because you only ever saved the part worth keeping.

But it does not stop there. A handoff, or a compacted summary, is built for one specific thread — usually to get one person or one task to the next step, and it tends to fade in usefulness once that step is done. A saved NeatContext context has no such expiry. It stays useful for the tenth unrelated issue that happens to touch the same system, for the teammate who was never part of the original conversation, and for the version of you who has forgotten the details six months from now. Handoff and compacting get you to the next message in one conversation. NeatContext gets you, and everyone on your team, to every future conversation after that.

## What NeatContext does about it

NeatContext pulls the useful knowledge out of a Claude Code conversation and saves it as a small, reusable package. It has two parts: a profile with the rules and habits that shaped how Claude worked, and a knowledge folder with the guides, findings, and notes worth keeping. It is not the whole chat — just the part that will actually help next time.

Here's how that compares to what Claude Code already offers:

| | Best for | What you get |
|---|---|---|
| **NeatContext** | Reusing knowledge in a new chat, or sharing it with your team | A short profile plus a knowledge folder — ready to reuse, not a full transcript |
| **Claude Code resume** | Picking up the same conversation again | The original chat and its full history |
| **Save or export a conversation** | Keeping a record | The full transcript, including everything that was said |

Once you save a context, you can connect it in a new chat, or hand it to a teammate. Claude then starts already knowing what your earlier, time-consuming conversation figured out.

## What it looks like in practice

Say you just finished tracking down a tricky bug: some order updates were slow on one partition, even though the overall numbers looked fine. Finding it took checking traces, per-partition delays, and worker logs. Instead of letting all that work disappear when you close the tab, you save it:

```text
You: /neatcontext:save event-partition-investigation

Claude:
Lite context folder: <folder>
Profile path: <folder>/profile.md
Knowledge folder: <folder>/knowledge
Use command: /neatcontext:use event-partition-investigation
```

Weeks later, a similar but different problem shows up — shipment updates are slow, but the overall numbers look normal again. You connect the saved context, and Claude picks up where the last investigation left off instead of starting blind:

```text
You: /neatcontext:use event-partition-investigation

Claude: Connected to event-partition-investigation.

You: Shipment updates are delayed, but overall queue lag is low. Help me investigate.

Claude: I will start with the checks from the saved context: per-partition lag,
        event size, partition keys, and deserialization time.
```

That's the whole idea: the second investigation starts with what the first one already learned, instead of everyone re-explaining it or digging through old chats.

## The commands

The plugin adds a small set of simple commands to Claude Code:

- **`/neatcontext:save [name]`** — save the useful work from the current chat as a new context.
- **`/neatcontext:use [name or number]`** — connect a saved context to the current chat.
- **`/neatcontext:list`** — show all the contexts you can connect to.
- **`/neatcontext:status`** — show what's connected right now, and flag any problems.
- **`/neatcontext:create`** — build a new context from an existing folder of notes, without needing a chat to pull it from.
- **`/neatcontext:import [folder]`** — add a context that a teammate shared with you.
- **`/neatcontext:delete [name or number]`** — remove a context you no longer need.
- **`/neatcontext:mode [auto|ask|manual]`** — choose whether Claude switches contexts on its own, asks first, or waits for you to say so.

## Install it

```bash
claude plugin marketplace add https://github.com/XTSoftwareLabs/neatcontext-plugins.git
claude plugin install neatcontext@neatcontext --scope user
```

Then restart Claude Code. (If you're already inside a session, you can run the same steps with `/plugin marketplace add` and `/plugin install`.) You'll need Claude Code 2.1.196 or later, and Node.js 18 or later. The NeatContext desktop app is only needed if you also want the more advanced team contexts it manages.

## Where your knowledge is stored

Everything a saved context needs — the profile and the knowledge folder — is stored locally on your computer, under `~/.neatcontext/lite`. The plugin only runs the Node.js code included in the repository, and it does not make outbound internet requests on its own. Its desktop app connection talks to NeatContext over `127.0.0.1` (your own machine). You can read the full details in the project's [privacy policy](https://github.com/XTSoftwareLabs/neatcontext-plugins/blob/main/PRIVACY.md).

## Why this matters beyond one person

Save one investigation and you save yourself some time later. Do it regularly, across a whole team, and something bigger builds up: a team library, made entirely out of the AI conversations people were already having anyway. Every debugging session, every planning chat, every "why does this service behave like that" conversation becomes a small, saved piece of knowledge instead of something that disappears. Import a teammate's saved context with `/neatcontext:import`, and their hard-won knowledge becomes yours too — no meeting required, no wiki page to write from scratch.

That library only grows richer over time, and it grows without extra work, because it comes from conversations your team was going to have anyway. This is a smaller, more personal version of a problem we've written about before: [why more context is not always better context](https://blog.neatcontext.com/ai/2026/07/14/why-more-context-is-not-always-better-context/) and [how to build efficient context for your AI client](https://blog.neatcontext.com/guide/2026/07/22/how-to-build-efficient-context-for-ai-clients/). Good context needs rules, knowledge, and a way to pull in current facts — and one of the best sources of real, team-specific knowledge is the work your team has *already done* with an AI. NeatContext keeps that knowledge around instead of throwing it away, and turns everyday conversations into something the whole team can keep using.

## Try it

The project is open source and free to use under the MIT license: [github.com/XTSoftwareLabs/neatcontext-plugins](https://github.com/XTSoftwareLabs/neatcontext-plugins). Install it, run `/neatcontext:save` after your next real debugging session, and see what it's like to have Claude start your next chat already knowing what the last one learned. We would love your feedback.
