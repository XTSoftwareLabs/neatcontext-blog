---
layout: post
title: "Why We Stopped Using an Automated SRE Agent"
date: 2026-07-12
categories: operations
---

For the past year, our platform engineering team chased the ultimate infrastructure dream: a fully autonomous, AI-driven Site Reliability Engineering (SRE) agent. We integrated it into our monitoring pipelines, granted it read access to our logs, and gave it a loose mandate to triage incidents while we slept.

We envisioned a future with zero on-call fatigue, immediate mean time to resolution (MTTR), and clean, hands-off infrastructure management.

Instead, we got a confident hallucination engine.

After months of tuning, debugging prompts, and dealing with 3:00 AM outages made worse by AI-driven confusion, we turned it off. Here is why we abandoned the automated SRE agent hype—and the paradigm shift we adopted instead.

## The Three Fatal Flaws of Autonomous SRE Agents

- **The "Haystack" Firehose & Signal Dilution**

    When an incident occurs in a modern cloud-native environment, systems emit an overwhelming stream of telemetry. A single microservice failure can trigger database connection spikes, network latency alerts, and cascading downstream drop-offs.

    Our automated agent was designed to ingest everything dynamically. When a high-severity alert triggered, it scraped thousands of lines of raw, unfiltered logs, metrics, and cloud traces across the cluster.

    **The Reality:** The agent experienced acute information overload. Instead of identifying the root cause, it struggled with signal dilution. It would link a completely unrelated database connection blip from twelve hours prior to a localized payment-gateway timeout happening right now. The agent didn't lack data; it lacked the human intuition required to ignore the noise.

- **The Illusion of RAG (Retrieval-Augmented Generation)**

    To help the agent understand our environment, we connected it via RAG to our internal documentation, deployment timelines, and post-mortem wikis.

    **The Reality:** AI is only as accurate as your stale documentation. In a fast-moving engineering organization, system boundaries shift daily. A network routing rule changed last Tuesday, but the internal runbook wasn't updated until Friday. The agent faithfully retrieved the old document and diagnosed the problem based on stale assumptions. The agent didn’t hallucinate the system architecture; the source material itself was wrong.

- **The "Confused Deputy" Risk**

    As SRE tools evolve, there is pressure to give them write-access—allowing them to execute CLI commands, restart Kubernetes pods, or rollback deployments autonomously.

    **The Reality:** Without strict boundaries, an agent lacks an abstention threshold. When faced with a novel edge case, it does not say "I don't know." It guesses. In a production environment, an incorrect guess isn’t a formatting bug; it is a catastrophic cascading failure. We quickly realized that keeping an agent completely autonomous was a liability we couldn't afford.

## The Paradigm Shift: Context is the key to succeed

Turning off the automated agent didn't mean abandoning LLMs. Large Language Models are incredibly capable at diagnosing software flaws, parsing dense logs, and analyzing stack traces. The problem wasn't the AI's "IQ"—the problem was the context delivery mechanism.

Instead of letting an automated platform-level agent crawl our environment and dynamically guess what matters, we realized engineers need a way to explicitly bundle, filter, and isolate localized data before it hits the LLM.

This is exactly why we shifted toward lightweight, desktop-first context utilities like [NeatContext](https://www.neatcontext.com).

To be clear about what this is: [NeatContext](https://www.neatcontext.com) is not another AI, and it does not contain an AI model. It connects locally to the AI client the engineer is already working in and hands that client a scoped context. The reasoning still happens where it always did; only the inputs change.

Instead of acting as a heavy background scraper, it lets on-call engineers build explicit Markdown-based domain profiles. When a specific service drops, the engineer connects a dedicated team context that scopes exactly which directories, local deployment configurations, error logs, and read-only tools are in play — an airtight boundary the assistant works inside of.

## Why a Lightweight, Local-First Approach Works:

- Strict System Boundaries: If our payment service is failing, the AI operates within an airtight sandbox—zero background noise, zero signal dilution. It only evaluates what the team profile explicitly scopes out.

- Injecting Tribal Knowledge: LLMs don't naturally know cross-team boundaries or custom architecture quirks. Explicit context bundling allows teams to inject localized rules (e.g., "If the Payment Team hits an error here, the deployment data shows it must be escalated to the Infra Team").

- Safety by Design: A desktop context organizer is inherently read-only. It hands over paths and read-only tools — never write authority — and keeps a local record of what was actually served. The assistant produces the diagnostic report or runbook suggestion; the engineer remains the critical "human-in-the-loop," executing the final validated commands.

- No New Trust Decision: The context layer contains no AI model and stores no LLM credential, so adopting it does not mean sending your incident data somewhere new. It reaches the same assistant your organization already approved.

## Moving Forward

The promise of a fully automated, hands-off SRE agent is an attractive illusion. The reality of infrastructure is too volatile, fragmented, and heavily reliant on tribal, team-specific domain knowledge.

We stopped trying to give AI an active footprint in our production clusters. By moving away from automated scrapers and switching to targeted context assembly, we slashed our MTTR, eliminated agent hallucinations, and finally gave our on-call engineers an assistant they could actually trust.
