# Agentic Engineering Resources

Writing and practical resources on how to build, run, and secure AI agents. Organized primarily by **topic**, not by resource format. Feedback is welcome.

## Agent engineering

**[Agentic Engineering Guide](./AgenticEngineeringGuide.md)** — a technical guide to terminal-native local agents and their engineering patterns, including agent behavior, tool-using environments, Claude Code, context management, spec-driven engineering, and GSD. The guide remains at the repository root to preserve existing links and its supporting scripts and planning references.

- [What Deserves to Become an Agent Skill](<agent-engineering/What Deserves to Become an Agent Skill.md>) — when a repeatable task justifies a reusable skill, and how to assess its added value.
- [Why Organizations Struggle to Get Agentic Projects from Demo to Shipped](<agent-engineering/Why Organizations Struggle to Get Agentic Projects from Demo to Shipped.md>) — operational, evaluation, and data challenges in shipping agents.

## Agent infrastructure

- [Enterprise Web Search Infrastructure for AI Applications](<agent-infrastructure/Enterprise Web Search Infrastructure for AI Applications.md>) — vendor, privacy, commercial, and architecture assessment of enterprise web-search infrastructure.

## Knowledge systems

- [When Does an AI Agent Need a Knowledge Graph?](<knowledge-systems/When Does an AI Agent Need a Knowledge Graph.md>) — informal knowledge structures, explicit semantics, and when additional structure is worthwhile.
- [Knowledge System Assessment skill](knowledge-systems/skills/knowledge-system-assessment/SKILL.md) — staged, read-only assessment of existing knowledge systems, duplication, and maintenance burden.

## Agent deployment safety

**[Agent deployment safety](agent-deployment-safety/README.md)** — a method for assessing and securing a particular deployment of an AI agent, focused on material harm:

- [Deployment parameters](agent-deployment-safety/docs/deployment-parameters.md) — seven properties shaping security requirements.
- [Internal research agent](agent-deployment-safety/docs/internal-research-agent.md) — application to a specific deployment.
- [Claude Code external security](agent-deployment-safety/docs/claude-code-external-security.md) — untrusted workload with a disposable sandbox and trusted gateways.

Work in progress; no implementation yet.

## Repository organization

Resources are grouped by their **primary subject**. A topic may contain articles, guides, research, methods, and skills. Each resource has one canonical location; cross-topic references should link to it rather than duplicate it.

The root holds navigation and repository-wide agent instructions. Existing guide-specific `.planning/` and `scripts/` material stays in its established locations because relocating it would disrupt working paths. Add new topic directories only when there is actual material for them.
