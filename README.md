# Agentic Engineering Resources

Writing on agentic engineering: how to build, run and secure AI agents. Feedback is welcome.

## The guide

**[AgenticEngineeringGuide.md](./AgenticEngineeringGuide.md)** — a technical guide about
terminal-native local agents and the engineering patterns around them:

- agentic behavior and agentic engineering basics
- local tool-using agent environments
- Claude Code as a concrete harness example
- context window management
- spec-driven engineering
- GSD as a workflow framework layered on top

The guide was written with the assistance of GSD on top of Codex CLI; the `.planning/` and
`scripts/` folders are the artifacts of that process.

## Articles

- [Enterprise Web Search Infrastructure for AI Applications](<articles/Enterprise Web Search Infrastructure for AI Applications.md>)
- [What Deserves to Become an Agent Skill](<articles/What Deserves to Become an Agent Skill.md>)
- [Why Organizations Struggle to Get Agentic Projects from Demo to Shipped](<articles/Why Organizations Struggle to Get Agentic Projects from Demo to Shipped.md>)

## Agent deployment safety

**[agent-deployment-safety/](agent-deployment-safety/README.md)** — a practical method for
assessing and securing a specific deployment of an AI agent, focused on material harm
(security, privacy, legal, financial, operational):

- [Deployment parameters](agent-deployment-safety/docs/deployment-parameters.md) — seven properties that drive a deployment's security requirements
- [Internal research agent](agent-deployment-safety/docs/internal-research-agent.md) — the method applied to a concrete deployment, comparing a purpose-built Python agent with Claude Code
- [Claude Code external security](agent-deployment-safety/docs/claude-code-external-security.md) — treating the agent as an untrusted workload: a disposable sandbox with narrow, trusted gateways

Work in progress; no implementation yet.
