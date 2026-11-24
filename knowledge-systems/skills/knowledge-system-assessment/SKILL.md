---
name: knowledge-system-assessment
description: Research and evaluate an existing knowledge system, its semantic structure, usability, maintenance burden, and whether improvements such as vocabularies, entity registries, or knowledge graphs are justified. Evidence-based and technology-neutral.
---

# Knowledge System Assessment

## Purpose

Evaluate an **existing** knowledge system before recommending new semantic structures, databases, indexes, or retrieval architectures.

The object may be a file repository, document library, knowledge base, database ecosystem, connected set of SaaS services, agent memory system, or a mixture. Determine what exists and how well it supports its actual users and agents before prescribing solutions.

**A finding that no changes are necessary is valid.** A more explicit ontology or knowledge graph is never the default recommendation.

## Operating boundaries

- **Research only. Do not implement changes.** Do not modify existing files, repositories, indexes, configurations, databases, or connected applications; do not create commits, issues, or persistent outputs. This restriction applies even if an inspected repository includes standing instructions to commit changes.
- Read-only inspection, searching, and analysis are permitted. If a task requires working files, use only isolated temporary storage outside the inspected system. Reports go in the conversation, not into the system under assessment.
- Do not delete, migrate, consolidate, or reorganize information. A recommendation to do so is not authorization.
- Work **one stage at a time**. After each stage, present a grounded report, state uncertainties, and **stop for explicit approval** before advancing. Approval authorizes only the next research stage, never implementation.
- Adapt investigation depth and examples to the system's purpose, scale, and available access. Do not assume any specific tooling or directory scheme.
- Avoid accessing material irrelevant to the evaluation. Respect the access restrictions of the environment.
- Distinguish *observed*, *documented*, *inferred*, and *unknown* findings. Support conclusions with specific paths, records, or other inspectable evidence when possible.

## Core assessment principles

1. **Establish a baseline first.** Assess existing maintenance, navigation, and consistency costs before proposing improvements.
2. **No duplication by default.** For every possible new structure, ask whether it copies facts or definitions already maintained elsewhere, creates competing authorities, or needs synchronization. Prefer pointers to authoritative sources and regenerable views over independently maintained copies.
3. **Semantics may already exist informally.** Folder hierarchy, naming conventions, labels, READMEs, data dictionaries, indexes, and cross-references can provide useful taxonomy, conceptual definitions, and navigable relationships without formal ontology or graph infrastructure.
4. **Distinguish needs from technologies.** Assess demonstrated failures in identity, discovery, cross-system interpretation, completeness, retrieval, or maintenance before evaluating RDF, OWL, graph databases, GraphRAG, or other implementations.
5. **Keep relationships selective.** Do not assume every document, email, transaction, or event should be modeled as a graph node or edge. A separate relationship registry needs evidence that it adds value beyond the structures that already exist.
6. **Minimize recurring work.** Evaluate total cost of ownership, including maintenance of identities, taxonomy, documentation, generated indexes, and explicit links.
7. **Scope recommendations to the system.** The best result for a personal repository may differ from an enterprise catalog or regulatory knowledge base.

## Workflow

### Stage 1 — Current system and maintenance baseline

**Goal:** Understand the existing architecture and the effort required to maintain it, without proposing a new design yet.

Inspect available architecture documentation, source organization, naming conventions, indexes, READMEs, logs, workflows, connected systems, and instructions used by people or agents.

Investigate:
- Where information resides; how it is classified, located, added, updated, and retired.
- What documentation, references, and indexes are authoritative, generated, or supporting.
- Who or what maintains each component; what changes require updates in several places.
- Existing duplication, synchronization points, fragility, and proven navigation shortcomings.
- Human supervision, agent work, tool use, and operational costs, where evidenced.
- Which claims are documented expectations versus observed practice.

**Deliverable:** Architecture overview, current maintenance workflow and cost baseline, strengths, verified problems, uncertainties, and evidence. Do **not** design an ontology or registry yet.

**Stop and ask for approval to proceed to Stage 2.**

### Stage 2 — Existing knowledge representation

**Goal:** Identify explicit and implicit semantic structure and assess how reliably it helps locate and connect information.

Look for:
- Controlled terms, glossary definitions, classifications, taxonomies, and conceptual models.
- Existing entity types, aliases, stable identifiers, references, and cross-system identity mappings.
- Explicit and implicit relationships through hierarchy, links, metadata, indexes, and documentation.
- Information that is directly accessible, reasonably inferable, ambiguous, or hard to discover.
- Real limitations versus merely missing formal terminology.

**Deliverable:** An evidence-backed map of existing semantic structures; identity and relationship mechanisms; effective areas, ambiguities, and actual gaps.

**Stop and ask for approval to proceed to Stage 3.**

### Stage 3 — Requirements, gaps, and candidate improvements

**Goal:** Determine which observed shortcomings materially affect actual use, then compare minimal solutions.

Evaluate candidate approaches only when relevant, such as:
- Keep the system unchanged.
- Improve existing documentation, conventions, or indexes.
- Add a shared vocabulary or lightweight domain definitions.
- Add stable identifiers or a minimal entity registry.
- Embed a few high-value relationships inside existing entity records.
- Add separate explicit relationships, a formal ontology, a knowledge graph, or improved retrieval tooling when justified.
- Use a different approach if the system warrants it.

Evaluate whether links already discoverable from existing structure need any extra representation. Do not require a dedicated relationship store.

**Deliverable:** Prioritized, evidenced gaps; alternatives with their expected benefit, implementation effort, flexibility, and disadvantages. Clearly note where evidence is insufficient.

**Stop and ask for approval to proceed to Stage 4.**

### Stage 4 — Duplication, authority, and maintenance analysis

**Goal:** Test credible options against the baseline, focusing particularly on long-term costs.

For each viable option:
- Identify current authoritative sources and which proposed fields would be new versus copied.
- Assess multiple sources of truth, drift, synchronization, and name/identifier changes.
- Consider whether references, generated views, or extensions to existing structures would suffice.
- Consider how entities, relations, and classifications would be created, corrected, and deprecated.
- Compare ongoing human effort, agent effort, review burden, and operational complexity.
- Identify any information that could *eventually* be consolidated or replaced, without changing it now.

**Deliverable:** Source-of-truth map; duplication and drift risks; maintenance impact compared with Stage 1; revised comparison of viable options.

**Stop and ask for approval to proceed to Stage 5.**

### Stage 5 — Conclusion and decision

**Goal:** Give the smallest justified recommendation, including the option to do nothing.

Answer directly:
1. What currently works well, and what demonstrably does not?
2. Is new semantic structure necessary? If so, what minimum form?
3. Would a vocabulary, identity registry, embedded relationships, or separate graph add distinct value?
4. How would the proposal avoid duplicated facts and competing authorities?
5. What new ongoing maintenance would it require compared with the baseline?
6. What should be left unchanged?
7. Is the expected benefit worth the cost and uncertainty?

**Deliverable:** Concise executive decision, key supporting observations, rejected alternatives, limitations, and an optional *future* implementation outline. No implementation.

## Reporting expectations

At each stage use an appropriate compact structure:
- Findings, with references to the inspected system.
- Assessment and implications.
- Unknowns / constraints.
- Decision or questions for the next stage.
- Explicit approval request.

Prefer qualitative judgments to invented metrics. If timing, accuracy, effort, or coverage are not measured, say so. Do not claim experiments were performed when they were not.

## Begin

Start with **Stage 1 only**. Inspect what is accessible, establish the current maintenance baseline, report findings in the conversation, and stop for approval.
