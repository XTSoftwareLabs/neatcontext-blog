---
layout: post
title: "Why More Context Is Not Always Better Context"
date: 2026-07-14
categories: ai
---

A recent discussion raised a sensible idea for AI-assisted engineering: make the organization's code and domain material broadly available to the AI, then use a local model tailored to the domain.

The instinct behind this proposal is right. An AI system should not reason about a business-critical change from one source file and a short prompt. It needs domain knowledge.

The problem is the jump from **access to everything** to **put everything in context**. Those are not the same design.

Broad access can make useful evidence discoverable. Indiscriminate inclusion can bury that evidence, mix incompatible sources, expose unnecessary data, and make a confident answer harder to audit. Running the model locally can improve privacy and control. Encoding domain knowledge into model weights still does not decide which facts are relevant, current, or authoritative.

The practical goal is not maximum context. It is the smallest defensible context for the task.

## An entire codebase is not one coherent specification

A repository contains more than the system's current intent. It may include deprecated modules, experimental branches, generated files, migration shims, old feature flags, example configurations, vendored dependencies, and tests that document behavior more accurately than the comments do.

If all of it is presented as equally important, the model has to infer which parts are live and which are historical. Similar names in unrelated services can look relevant. An obsolete implementation may be easier to find than the replacement. A test fixture can be mistaken for a production default.

Long context windows do not remove this problem. A [2026 TACL study of long-context position bias](https://aclanthology.org/2026.tacl-1.15/) found that models systematically over-relied on particular parts of the input. The bias worsened as context length increased and persisted across prompt formats and task types. Generating more reasoning paths through self-consistency did not fix the issue; in the study, it made long-context performance worse.

Repository-level code research points in the same direction. A 2026 study on [context pruning for coding agents](https://arxiv.org/abs/2605.15315) found that much of the code read by agents was irrelevant to the task. Its pruning method matched or outperformed full-context baselines in 12 of 16 multi-turn comparisons and reduced token use. One study does not prove that aggressive pruning is always best, but it supports a more useful principle: repository-wide availability is valuable, while task-level selection is still necessary.

The AI may need access to the whole repository. It rarely needs the whole repository in a single working set.

## Requirements accumulate contradictions

Business and technical requirements are not a clean second layer that can simply be added to the code.

Real organizations keep approved specifications beside abandoned proposals, meeting notes, work items, architecture decisions, support exceptions, and rollout plans. A business rule may describe the desired future state while production code still implements the current state. A technical document may have been correct before a later decision changed the boundary between two services.

Giving the model every version does not tell it which version governs the task. Worse, the model may smooth over the disagreement and produce a plausible compromise that nobody approved.

Good context should preserve conflicts instead of hiding them. Each source needs basic provenance: owner, status, version or commit, effective date, and why it was selected. When two authoritative sources disagree, the output should identify the conflict and ask for a decision rather than quietly resolving policy through text prediction.

## Sensitive documents make over-collection a real risk

Domain knowledge often includes material that should not be exposed by default: customer records, support conversations, incident reports, security findings, contracts, internal financial data, confidential roadmaps, and employee information. Some of those documents may be essential to a particular investigation. That does not make all of them appropriate context for every request.

Classification has to happen before assembly. A [2026 NIST draft on data-classification practices](https://csrc.nist.gov/News/2026/sp-1800-39-ipd-data-classification-practices) emphasizes discovering, identifying, and labeling sensitive unstructured data so organizations can reduce the risk of losing or mismanaging it. A context system that treats every accessible document as interchangeable removes exactly the distinctions needed to apply that control.

The risk is not limited to disclosure. AI systems may also alter important documents incorrectly. The 2026 [DELEGATE-52 study](https://arxiv.org/abs/2604.15597) tested 19 models across long document-editing workflows. Even the frontier models tested corrupted an average of 25% of document content by the end, and performance worsened with larger documents, longer interactions, and distractor files. The benchmark is a simulation rather than a production incident log, but its result argues for read-only source context, reviewable outputs, and human approval before changing a system of record.

Local processing reduces disclosure outside the organization's boundary, which is one reason we prefer it when practical. It does not remove the need for least-privilege access inside that boundary. A locally running model can still receive a customer record that the user did not need, preserve sensitive text in logs, or include confidential details in its output.

For sensitive work, the context should contain only the necessary source material, retain its classification and provenance, and produce an output that an accountable reviewer can verify.

## Run locally when practical—but keep changing knowledge outside the model

Running a model locally is a strong choice. It can keep sensitive material on controlled infrastructure, reduce dependence on an external service, and allow teams to choose a model that fits their hardware and latency needs. When an organization has the hardware and operational capacity, we think local inference should be preferred—especially for sensitive code and internal knowledge.

That is separate from embedding the organization's knowledge into the model's weights.

Doing so takes a snapshot of today's code, requirements, procedures, support playbooks, or product assumptions and embeds patterns from that snapshot into model weights. When a requirement changes next week, the old knowledge does not become visibly obsolete. There is no source file to diff, review date to check, or citation to follow. The team must identify the change, rebuild training data, update the model, evaluate for regressions, and redeploy. Until then, the model can confidently apply stale knowledge.

Changing facts and rules should remain in versioned sources and enter the model as runtime context.

A 2026 study of [lifelong knowledge editing](https://www.sciencedirect.com/science/article/pii/S0020025526007796) describes the core problem directly: knowledge in a deployed model remains static, while frequent parameter updates are costly. The paper proposes a more efficient editing method, but it also documents how repeated weight changes can accumulate interference and distribution drift. Weight updates are not equivalent to updating a document store. For changing organizational knowledge, fresh, inspectable sources should determine what the model knows about the current situation.

A useful division of responsibility is:

- use a local model for language, patterns, and reasoning when practical;
- use selected, versioned sources for current facts and constraints;
- use tools and tests for verification;
- keep accountable decisions with people.

The same context discipline should apply whether the model runs on a laptop, in a private cluster, or behind a public API.

## More access also creates a larger trust boundary

Code, documentation, tickets, support transcripts, and imported files are inputs, not instructions. An AI agent may not reliably preserve that distinction.

[NIST's 2026 work on securing AI agent systems](https://www.nist.gov/news-events/news/2026/01/caisi-issues-request-information-about-securing-ai-agent-systems) identifies interaction with adversarial data, including indirect prompt injection, as a distinct risk. A repository-wide context source increases the number of artifacts that must be treated as untrusted. A comment, README, generated file, support transcript, or imported issue can become part of the model's working context without being relevant to the user's task.

This does not mean an AI tool should never search broadly. It means discovery should not imply trust, and read access should not imply permission to act. Context sources need boundaries, sensitive operations need separate authorization, and retrieved content should retain its origin.

## Build a context packet, not a context dump

Consider a request to change a refund workflow for one product. A context dump might include every service, every product requirement, years of support conversations, customer records, and unrelated contracts.

A task-specific context packet would be smaller and more explicit:

- the request, affected product, and decision to be made;
- the active refund policy and its owner;
- the relevant service entry points, domain types, tests, and recent changes;
- a small, redacted set of relevant support examples, if they are necessary;
- known disagreements or unanswered questions;
- excluded sources and the reason they were excluded;
- expected validation, such as tests and review by the product or data owner.

The wider sources remain available for search. They are promoted into the working packet only when there is a reason to include them. The packet can change as the investigation reveals a dependency, but every addition is visible.

This approach has several practical properties:

1. **Scope is explicit.** The team can see what the model was allowed to consider.
2. **Sources remain traceable.** Claims can point back to a file, commit, requirement, or document owner.
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

The original proposal correctly recognizes that AI needs code, requirements, and internal knowledge. The adjustment is architectural: give the system a way to find what it is allowed to use, but do not ask the model to reason over everything at once.

More context expands the search space. Better context makes an answer defensible.
