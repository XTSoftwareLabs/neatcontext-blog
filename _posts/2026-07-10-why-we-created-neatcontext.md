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

The idea is simple:

**Make it easy to give LLMs the right operational context, without forcing teams to centralize all their knowledge into another platform.**

With NeatContext, every team can have its own domain knowledge. A service team can keep its own troubleshooting guides, runbooks, notes, logs, and relevant documents. Another team can organize things differently. During an incident, engineers can adjust the context based on what they are investigating.

The context can evolve naturally with the team.

## Bring your own systems

Operational knowledge rarely lives in one place.

That is why NeatContext is designed around extensions. Extensions can help connect to the systems you already have, whether they are public tools, internal platforms, knowledge base systems, incident management systems, work item trackers, or service-specific APIs.

You can choose where your domain knowledge comes from.

It may come from a Git repository that your team already maintains. It may come from an internal documentation system that syncs locally. It may come from files on your machine. It may come from an internal incident system. It may come from a custom extension that pulls the exact data your team needs.

NeatContext does not try to own all of this knowledge.

Instead, it helps you assemble the right context when you need it.

## You control how knowledge is shared with AI

Different companies have different requirements for AI usage.

Some teams are comfortable using a public API endpoint for LLM access. Some teams want to route requests through a company gateway. Some teams prefer to use a local model so sensitive knowledge stays entirely within their own environment.

NeatContext is designed to support that flexibility.

You decide what LLM endpoint to use. You decide what systems to connect. You decide what knowledge to include. You decide how much context is shared and where it goes.

We believe this control matters, especially for operational work.

Incident handling often involves sensitive, fast-changing, and highly specific information. Teams should be able to use AI without giving up control over their domain knowledge.

## No heavy deployment

We also wanted NeatContext to be easy to start using.

Many operational tools require platform-level setup, admin approval, data migration, complex onboarding, or centralized deployment. That can be appropriate for some systems, but it can also slow down experimentation.

NeatContext is different.

It is a desktop app. You can install it, point it at the knowledge you care about, connect the systems you need, and start using it. A team does not need to wait for a large platform rollout before learning whether AI-assisted incident handling can actually help them.

We want the first step to be simple.

## A NeatContext example

The following example shows how NeatContext can guide incident response differently depending on the domain knowledge available to each team.

The same incident can lead to different recommendations for different teams. For the payment team, the right recommendation is not to keep debugging payment logic. The available context points to an infrastructure-owned problem, so the correct response is to hand the incident off to the infrastructure team.

![NeatContext analysis for the payment team showing that the incident should be handed off to infrastructure]({{ "/assets/images/paymentteam.png" | relative_url }})

For the infrastructure team, the same incident leads to a different result. With infrastructure-specific knowledge, NeatContext can identify the operational action the team should take.

![NeatContext analysis for the infrastructure team showing an actionable infrastructure response]({{ "/assets/images/infrateam.png" | relative_url }})

This is why the right context matters. NeatContext is not only summarizing an incident; it is helping match the next step to the team, the system, and the situation.

## Our belief

We believe AI can meaningfully improve operations.

But we do not believe the answer is simply “add an LLM” to every incident platform.

The harder and more important problem is context.

The LLM needs the right knowledge, at the right time, for the right team, with the right level of control. That context must be flexible, local when needed, easy to update, and connected to the systems teams already use.

That is the reason we created NeatContext.

We are still early, and we are building this with real operational workflows in mind. Our goal is not to replace your incident management platform, your knowledge base, or your internal systems. Our goal is to help your team bring the right domain knowledge to AI so it can actually be useful when real incidents happen.

Give NeatContext a try: https://www.neatcontext.com

We would love to hear your thoughts, feedback, and ideas.
