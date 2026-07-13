---
layout: post
title: "Why More Context Is Not Always Better Context"
date: 2026-07-14
categories: ai
---

A recent discussion raised a sensible idea for AI-assisted engineering: make the organization's code and domain material broadly available to the AI, then use a local model tailored to the domain.

The instinct behind this proposal is right. An AI system should not reason about a business-critical change from one source file and a short prompt. It needs domain knowledge.

The problem is the jump from **access to everything** to **put everything in context**. Those are not the same design.

Broad access can make useful evidence discoverable. Indiscriminate inclusion can bury that evidence, mix incompatible authorities, expose unnecessary data, and make a confident answer harder to audit. Running the model locally can improve privacy and control. Fine-tuning that model with domain knowledge still does not decide which facts are relevant, current, or authoritative.

The practical goal is not maximum context. It is the smallest defensible context for the task.

## An entire codebase is not one coherent specification

A repository contains more than the system's current intent. It may include deprecated modules, experimental branches, generated files, migration shims, old feature flags, example configurations, vendored dependencies, and tests that document behavior more accurately than the comments do.

If all of it is presented as equally important, the model has to infer which parts are live and which are historical. Similar names in unrelated services can look relevant. An obsolete implementation may be easier to find than the replacement. A test fixture can be mistaken for a production default.

Long context windows do not remove this problem. The 2025 ICML paper [*NoLiMa: Long-Context Evaluation Beyond Literal Matching*](https://proceedings.mlr.press/v267/modarressi25a.html) evaluated 13 models advertised with context windows of at least 128K tokens. Performance fell significantly as context grew when finding the answer required inference rather than a literal text match. At 32K tokens, 11 models scored below half of their strong short-context baselines.

Repository-level code research points in the same direction. A 2025 AAAI study on [hierarchical context pruning](https://ojs.aaai.org/index.php/AAAI/article/view/34782) reduced prompts from roughly 50,000 repository tokens to around 8,000. Its pruned prompts improved completion accuracy on five of the six code models tested while also improving throughput. One benchmark does not prove that aggressive pruning is always best, but it supports a more useful principle: repository-wide availability is valuable, while task-level selection is still necessary.

The AI may need access to the whole repository. It rarely needs the whole repository in a single working set.

## Requirements accumulate contradictions

Business and technical requirements are not a clean second layer that can simply be added to the code.

Real organizations keep approved specifications beside abandoned proposals, meeting notes, work items, architecture decisions, support exceptions, and rollout plans. A business rule may describe the desired future state while production code still implements the current state. A technical document may have been correct before a later decision changed the boundary between two services.

Giving the model every version does not tell it which version governs the task. Worse, the model may smooth over the disagreement and produce a plausible compromise that nobody approved.

Good context should preserve conflicts instead of hiding them. Each source needs basic provenance: owner, status, version or commit, effective date, and why it was selected. When two authoritative sources disagree, the output should identify the conflict and ask for a decision rather than quietly resolving policy through text prediction.

## Laws and regulations are not just another document folder

Legal material adds jurisdiction, effective dates, amendments, definitions, exceptions, regulatory guidance, and an order of authority. The applicable rule may depend on where a user resides, what data is processed, the role of the organization, or when an event occurred. A superseded rule can look almost identical to the current one.

Even current systems built specifically for legal research do not make source selection disappear. The 2026 [Legal RAG Bench](https://arxiv.org/abs/2603.01710) evaluated Gemini 3.1 Pro and GPT-5.2 on complex questions about Victorian criminal law. It found retrieval to be the primary driver of end-to-end performance, with the choice of LLM having a more moderate effect on correctness and groundedness. Many apparent hallucinations began with retrieval failures. In other words, the reasoning model cannot reliably recover an authority that the context process failed to select.

Other recent evidence is encouraging but still argues for careful scope. A [2026 article reporting a randomized controlled trial](https://repository.law.umich.edu/facarticles/3173/) found that a legal RAG tool and a reasoning model improved the quality and productivity of upper-level law students across a defined set of legal tasks. That is a strong case for grounded AI assistance. It is not a case for treating an unfiltered collection of global law as a self-applying rulebook.

There is also a governance problem in sending everything. If source material contains personal data, indiscriminate inclusion may work against current regulatory guidance. The French data protection authority's [2026 recommendations for AI systems](https://www.cnil.fr/en/ai-system-development-cnils-recommendations-to-comply-gdpr) call for using personal data that is relevant and necessary for the defined objective, documenting its sources, and keeping that documentation current. Local processing can reduce disclosure to an external provider, but local does not automatically mean appropriately scoped.

For high-impact decisions, an AI response should be an analysis aid, not the final legal determination. The context should contain the relevant primary text, its jurisdiction and effective date, and links that a qualified reviewer can verify.

## Run locally when practical—but keep changing knowledge outside the model

Running a model locally is a strong choice. It can keep sensitive material on controlled infrastructure, reduce dependence on an external service, and allow teams to choose a model that fits their hardware and latency needs. When an organization has the hardware and operational capacity, we think local inference should be preferred—especially for sensitive code and internal knowledge.

That is separate from specializing or fine-tuning the model with the organization's knowledge.

Fine-tuning takes a snapshot of today's code, requirements, policies, or regulations and embeds patterns from that snapshot into model weights. When a requirement changes next week, the old knowledge does not become visibly obsolete. There is no source file to diff, effective date to check, or citation to follow. The team must identify the change, rebuild training data, tune again, evaluate for regressions, and redeploy. Until then, the model can confidently apply stale knowledge.

Fine-tuning can still be useful for stable behavior: terminology, output structure, tool-use patterns, or a narrowly defined task. Frequently changing facts and rules should remain in versioned sources and enter the model as runtime context.

A [2025 comparison of long-context and retrieval approaches](https://arxiv.org/abs/2503.16071) found that long context was better suited to some global questions, while retrieval was more competitive for questions targeting specific information. The researchers then combined retrieval-informed data generation with tuning a smaller model. The result is a useful reminder that tuning and retrieval solve different problems and can complement each other. Tuning should shape how a model works; fresh, inspectable sources should determine what it knows about the current situation.

A useful division of responsibility is:

- use a local model for language, patterns, and reasoning when practical;
- use selected, versioned sources for current facts and constraints;
- use tools and tests for verification;
- keep accountable decisions with people.

The same context discipline should apply whether the model runs on a laptop, in a private cluster, or behind a public API.

## More access also creates a larger trust boundary

Code, documentation, tickets, and downloaded legal material are inputs, not instructions. An AI agent may not reliably preserve that distinction.

[NIST describes agent hijacking](https://www.nist.gov/news-events/news/2025/01/technical-blog-strengthening-ai-agent-hijacking-evaluations) as an indirect prompt-injection attack in which malicious instructions are placed in data an agent ingests, causing unintended actions. A repository-wide context source increases the number of artifacts that must be treated as untrusted. A comment, README, generated file, or imported issue can become part of the model's working context without being relevant to the user's task.

This does not mean an AI tool should never search broadly. It means discovery should not imply trust, and read access should not imply permission to act. Context sources need boundaries, sensitive operations need separate authorization, and retrieved content should retain its origin.

## Build a context packet, not a context dump

Consider a request to change refund eligibility for customers in one market. A context dump might include every service, every product requirement, and a library of global regulations.

A task-specific context packet would be smaller and more explicit:

- the request, affected market, and decision to be made;
- the active refund policy and its owner;
- the relevant service entry points, domain types, tests, and recent changes;
- the applicable primary legal or regulatory sections, with jurisdiction and effective date;
- known disagreements or unanswered questions;
- excluded sources and the reason they were excluded;
- expected validation, such as tests and review by the policy or legal owner.

The wider sources remain available for search. They are promoted into the working packet only when there is a reason to include them. The packet can change as the investigation reveals a dependency, but every addition is visible.

This approach has several practical properties:

1. **Scope is explicit.** The team can see what the model was allowed to consider.
2. **Sources remain traceable.** Claims can point back to a file, commit, requirement, or authority.
3. **Freshness can be checked.** Time-sensitive material carries a version or effective date.
4. **Contradictions remain visible.** The system can report disagreement instead of inventing consensus.
5. **Sensitive data is minimized.** Only material necessary for the task enters the working set.
6. **The model is replaceable.** The same reviewed packet can be used with a local or hosted model.

## The approach we are taking

This is the context model we have been exploring with [NeatContext](https://www.neatcontext.com): keep knowledge close to the teams and systems that own it, assemble a bounded working set for a specific task, and let the engineer decide what is included before it reaches an LLM.

The important part is not the product name. It is the separation of concerns:

- broad sources for discovery;
- a narrow, reviewable packet for reasoning;
- a local model where practical, or a controlled hosted model where necessary;
- human review for consequential decisions.

The original proposal correctly recognizes that AI needs code, requirements, and regulatory knowledge. The adjustment is architectural: give the system a way to find everything, but do not ask the model to reason over everything at once.

More context expands the search space. Better context makes an answer defensible.
