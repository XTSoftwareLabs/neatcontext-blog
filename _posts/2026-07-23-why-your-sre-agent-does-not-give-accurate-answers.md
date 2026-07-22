---
layout: post
title: "Why Your SRE Agent Doesn't Give Accurate Answers — and How to Fix It"
date: 2026-07-23
categories: operations
---

If your team has tried an AI-powered SRE agent, you have probably felt the gap. The demo was impressive. Then a real incident happened, you asked it what was going on, and the answer was confident, plausible, and not quite right. So the on-call engineer did what they always did: opened the dashboards, read the logs, checked the last deploy, and figured it out themselves — now with an extra step in the way.

The agent did not fail because it was a weak model. It failed because it did not know *your* systems.

## The real reason: it lacks your team's domain knowledge

A general-purpose SRE agent knows a great deal about software in the abstract. It can read a stack trace, explain a class of error, and describe common debugging patterns. What it cannot know is the part that actually resolves your incidents:

- that a particular alert is noisy unless it fires together with a second signal;
- that this service's 5xx spikes almost always trace back to a shared connection pool owned by another team;
- that the runbook says *never* fail over this primary while it is saturated but healthy;
- that last Tuesday's routing change moved a boundary the docs still describe the old way;
- that when the payments team sees this error, it must be escalated to infra rather than retried.

This is domain knowledge, and it is specific to your team, your architecture, your history, and your conventions. Without it, even a strong model reasons from generic assumptions. It produces an answer that sounds right and points the wrong way. During an incident, a wrong-but-confident answer is worse than no answer — the on-call engineer has to first disprove the agent, then start the real investigation. That is how an assistant meant to save time ends up costing it.

The accuracy problem is not fundamentally a model problem. It is a context problem.

## The knowledge isn't bound to the platform — it's everywhere

Most SRE agents ship as a feature of a larger platform: an incident management tool, an observability suite, an internal automation system. That framing quietly assumes the knowledge the agent needs lives inside the platform too.

It doesn't.

Your team's operational knowledge is scattered across everything you use. Some of it is in Git. Some is in an internal wiki or docs system. Some is in incident tickets, dashboards, spreadsheets, chat threads, architecture diagrams, and half of it is in the heads of the engineers who have been on call the longest. A meaningful part of it is temporary — a note someone writes during a rollout that matters for exactly one week.

A platform can only reason over what has been loaded into it. So the moment an agent lives inside a single platform, it is working with a slice of the picture — and usually not the slice that resolves the incident in front of you.

## "Connect your other tools" helps, but the model is still too heavy

Some platforms have recognized this and now offer integrations: connect your repos, your docs, your ticketing system, and so on. That is a real improvement, and it acknowledges the right problem.

But look at the shape of the solution. You are still asked to bind everything to the platform — to route each source through its connectors, keep those integrations maintained, manage access inside it, and accept that the platform sits in the middle of all your operational knowledge. You are consolidating your team's most sensitive, fast-changing knowledge into one more system whose job is to be the center of gravity.

For some organizations that trade-off is fine. For many, it is a heavy commitment made mostly to work around a single limitation: the agent can only use what its platform can see.

## Think deeper: what a company actually needs is context + AI clients

Step back from the platform framing and the problem gets simpler.

You do not actually need another platform to house an agent. Two ingredients get you an accurate answer during an incident:

1. **A capable AI client.** ChatGPT, Claude, Claude Code, Codex CLI — these are already very good at reading logs, parsing traces, connecting scattered facts, and drafting investigation steps. Your team may already be approved to use one. The reasoning engine is a solved and commoditized part of the stack.

2. **Your team's domain knowledge, as context.** This is the part no vendor can supply, because only your team and your company know it. It is the rules, the runbooks, the troubleshooting guides, and the ability to pull current state from the systems you rely on.

The powerful model is not the missing piece — you can already get one. The missing piece is a way to get *your* knowledge in front of that model, scoped to the incident at hand, without first migrating everything into a platform that wants to own it.

Put those two ingredients together and on-call engineers finally get the thing the SRE-agent demo promised: an assistant that reasons about the incident the way a senior teammate would, because it is working from the same knowledge they would have.

The question becomes practical: how do you build that context? We wrote up an approach in detail — [how to build efficient context for AI clients](https://blog.neatcontext.com/guide/2026/07/22/how-to-build-efficient-context-for-ai-clients/) — which breaks a good context into three parts: the **rules** that shape how the AI behaves, the **knowledge** that describes how your systems work, and the **retrieval** that pulls current state from the systems your team depends on.

## How NeatContext fits this scenario

This is exactly the problem [NeatContext](https://www.neatcontext.com) is built for. Instead of adding another platform with another agent inside it, NeatContext helps your team assemble the right context and hand it to the AI client you already use.

Here is how it changes the incident scenario above:

- **Your knowledge stays where it lives.** You organize rules as Markdown-based domain profiles and keep runbooks, TSGs, and notes as versioned local source. Nothing has to be migrated into a central platform to become useful.
- **Retrieval reaches the systems you rely on.** Read-only extensions connect the tools you already use — Git, internal docs, incident systems, custom APIs — so the AI can pull current state (the last deploy, the live error rate, the relevant log) instead of guessing from stale assumptions. Credentials stay in your OS keychain, not in the AI.
- **Context is scoped to the incident.** When the checkout service is failing, the on-call engineer connects the relevant team context. The assistant reasons inside that boundary — no unrelated services, no noise, no signal dilution — so the answer reflects *this* team's escalation rules and *this* service's failure modes.
- **The whole team shares the same context.** With the [team library](https://docs.neatcontext.com/features/library#the-team-library-optional-read-only), domain profiles, knowledge, and extensions are shared read-only across all team members, so the next engineer on call inherits the same context instead of rebuilding it from memory.

One point worth being precise about: **NeatContext does not contain an AI model.** It does not read your sources, rank them, or summarize them before your assistant sees them. It scopes the boundary and hands the selected profiles, folders, and read-only tools to the AI client the engineer is already using — which does the reading and reasoning itself. There is no new endpoint to approve and no LLM credential stored anywhere in it. The AI your organization already trusts is the one that answers; it just finally has your team's knowledge in front of it.

Same incident, same model, a very different answer — because the difference was never the model. It was the context.

## See it in action

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 1.5rem 0;">
  <iframe src="https://www.youtube.com/embed/lh5ZgP6CHss" title="NeatContext" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

## The takeaway

An SRE agent gives inaccurate answers when it is missing your team's domain knowledge — and binding that knowledge to yet another platform is a heavy way to solve a problem that does not require it. What your on-call engineers actually need is simpler: a capable AI client, which they likely already have, plus a disciplined way to give it the rules, knowledge, and retrieval that only your team can provide.

Build the context, hand it to the AI you already trust, and the accurate answer follows.

Give NeatContext a try at [neatcontext.com](https://www.neatcontext.com), or jump into the hands-on demo at [github.com/XTSoftwareLabs/neatcontext-demo](https://github.com/XTSoftwareLabs/neatcontext-demo). We would love your feedback.
