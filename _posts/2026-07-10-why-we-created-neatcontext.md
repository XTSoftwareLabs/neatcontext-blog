---
layout: post
title: "Why We Created NeatContext"
date: 2026-07-10
categories: product
---

AI is changing how engineering teams think about operations.

Incident management platforms, knowledge base tools, SRE agents, and automation systems are all trying to reduce the manual effort involved in keeping services reliable. The goal is a good one: help teams respond faster, reduce repetitive work, and make incident handling less painful.

We have been exploring the same direction.

Like many teams, we are excited about what LLMs can do for operational work. They can summarize logs, explain alerts, draft investigation steps, connect scattered pieces of information, and help engineers move faster during stressful moments.

But as we tried these new approaches, one thing became very clear:

**The real key is not just the LLM. The real key is whether the LLM has the right domain knowledge.**

Without the right context, even a very powerful model often gives generic answers. It may know common debugging patterns, but it does not know your service, your architecture, your deployment history, your team’s conventions, your runbooks, your past incidents, or the small details that actually matter during a real outage.

That is the problem NeatContext is built to solve.

## Incident handling depends on team-specific knowledge

In real operations, the same type of incident can mean very different things for different teams.

A high error rate may require one team to check a recent deployment. Another team may need to inspect a downstream dependency. Another team may know that a specific alert is noisy unless it appears together with a second signal. A troubleshooting guide that is useful for one service may be completely wrong for another.

This is why a general LLM is often not enough.

For real incident handling, the LLM needs domain knowledge that is specific to the team, the service, the environment, and the current situation. It needs access to the things engineers actually rely on: troubleshooting guides, runbooks, logs, dashboards, recent changes, work items, known issues, architecture notes, internal documents, and sometimes even temporary notes created during the incident itself.

This knowledge is not static. It changes all the time.

A runbook may be updated after a postmortem. A mitigation step may be removed after a system migration. A new dependency may change the investigation flow. A service owner may add a temporary note during a rollout. During an incident, an engineer may want to quickly add a log file, remove irrelevant context, or include a specific document for the LLM to reason over.

That kind of context needs to be flexible.

## Centralizing all domain knowledge is hard

Many existing platforms try to solve the problem by asking teams to store and maintain their domain knowledge inside a centralized system.

In theory, this sounds reasonable.

In practice, it is extremely difficult.

Domain knowledge is spread everywhere. Some of it lives in Git repositories. Some is in internal documentation systems. Some is in incident tickets, dashboards, spreadsheets, Slack discussions, architecture diagrams, or the memory of experienced engineers. Some information is long-term and official. Some is temporary and only useful for one investigation.

Trying to move all of that into a single platform, keep it clean, keep it updated, and make it useful for every team is a huge operational burden.

There is also another important concern: this knowledge is part of a company’s intellectual property. It may include internal architecture, operational processes, customer-impact information, sensitive logs, and details about how systems are built and maintained.

Teams should not have to copy everything into yet another centralized platform just to make AI useful.

We believe domain knowledge should stay close to the team that owns it. It should be easy to add, remove, update, and control. It should fit into the way teams already work, not force every team into a new centralized process.

## Generic agent tools are powerful, but often too broad

Another option is to use lightweight open-source agent frameworks or generic AI tools.

These tools can be powerful. They can call APIs, execute tasks, connect tools, and automate workflows. But for operational work, especially incident handling, they often start from a very generic place.

To make them useful for a real SRE or operations workflow, teams may need to spend significant time customizing prompts, building integrations, managing context, defining tools, controlling access, and shaping everything around their own incident process.

For some teams, that flexibility is useful. But for many teams, it becomes another project to maintain.

We wanted something more focused.

Something lightweight enough to start using quickly, but specialized enough to support real operational work.

That is why we created NeatContext.

## What NeatContext is

NeatContext is a desktop app designed for operational work and incident handling.

It runs on your local machine. Your domain knowledge stays local unless you choose otherwise. Teams can organize the knowledge they need, add or remove context during an investigation, and connect to the systems they already use through extensions.

One thing NeatContext deliberately is not: another AI to talk to. It has no chat window and no model of its own. You keep working in the AI client you already use — Claude Code, Codex CLI, Claude Desktop, or ChatGPT Desktop. NeatContext connects to it locally and hands it your team's context; your client reads, searches, and answers with its own model.

The idea is simple:

**Make it easy to give the AI you already use the right operational context, without forcing teams to centralize all their knowledge into another platform.**

With NeatContext, every team can have its own domain knowledge. A service team can keep its own troubleshooting guides, runbooks, notes, logs, and relevant documents. Another team can organize things differently. During an incident, engineers can adjust the context based on what they are investigating.

The context can evolve naturally with the team.

## Bring your own systems

Operational knowledge rarely lives in one place.

That is why NeatContext is designed around extensions. Extensions are read-only connectors to the systems you already have, whether they are public tools, internal platforms, knowledge base systems, incident management systems, work item trackers, or service-specific APIs. You pick which ones a given context may use, and their credentials stay in your OS keychain rather than being handed to the AI.

You can choose where your domain knowledge comes from.

It may come from a Git repository that your team already maintains. It may come from an internal documentation system that syncs locally. It may come from files on your machine. It may come from an internal incident system. It may come from a custom extension that pulls the exact data your team needs.

NeatContext does not try to own all of this knowledge.

Instead, it helps you assemble the right context when you need it.

## You control how knowledge is shared with AI

Different companies have different requirements for AI usage. Some standardize on one assistant. Some route everything through a company gateway. Some are still deciding.

NeatContext stays out of that decision. Because it runs no model, there is no NeatContext endpoint to approve, no model configuration to manage, and no credential for an LLM stored in the app. Whatever AI your team is already allowed to use remains the AI that answers.

You decide what systems to connect. You decide what knowledge to include. You decide which AI client receives it and how much of it is in scope for a given investigation. The context boundary is read-only: NeatContext hands over paths and read-only tools, never write access to your systems, and it keeps a local activity log of what was actually served.

We believe this control matters, especially for operational work.

Incident handling often involves sensitive, fast-changing, and highly specific information. Teams should be able to use AI without giving up control over their domain knowledge.

## No heavy deployment

We also wanted NeatContext to be easy to start using.

Many operational tools require platform-level setup, admin approval, data migration, complex onboarding, or centralized deployment. That can be appropriate for some systems, but it can also slow down experimentation.

NeatContext is different.

It is a desktop app. You can install it, point it at the knowledge you care about, connect the systems you need, and start using it. A team does not need to wait for a large platform rollout before learning whether AI-assisted incident handling can actually help them.

We want the first step to be simple.

## A NeatContext example

Consider one incident — elevated 5xx on a checkout API — and two teams asking the same AI client about it.

The payments engineer asks with their payments context connected. Their profile and runbooks say what payment-side saturation looks like and who owns the shared connection pool. The answer that comes back is not a longer debugging checklist: the evidence points at an infrastructure-owned problem, so the useful next step is a handoff to the infrastructure team with the incident link, the first pool-error log line, and the deploy timeline.

The infrastructure engineer asks the same question with their own context connected. Now the relevant knowledge includes the pool configuration history and a troubleshooting guide that explicitly forbids failing over a saturated-but-healthy primary. The answer names the specific change to revert and the action to avoid.

Same incident, same AI client, same underlying model. The difference is which team's knowledge was in scope — and, just as importantly, which knowledge was not.

This is why the right context matters. NeatContext is not summarizing the incident for you; it is making sure the AI you already trust is reasoning over the material your team would have handed a colleague.

## Our belief

We believe AI can meaningfully improve operations.

But we do not believe the answer is simply “add an LLM” to every incident platform.

The harder and more important problem is context.

The AI needs the right knowledge, at the right time, for the right team, with the right level of control. That context must be flexible, local when needed, easy to update, and connected to the systems teams already use.

That is the reason we created NeatContext.

We are still early, and we are building this with real operational workflows in mind. Our goal is not to replace your incident management platform, your knowledge base, your internal systems — or the AI assistant your team already works in. Our goal is to help your team bring the right domain knowledge to that assistant so it can actually be useful when real incidents happen.

Give NeatContext a try: [neatcontext.com](https://www.neatcontext.com)

You can also jump into the hands-on demo: [github.com/XTSoftwareLabs/neatcontext-demo](https://github.com/XTSoftwareLabs/neatcontext-demo)

We would love to hear your thoughts, feedback, and ideas.
