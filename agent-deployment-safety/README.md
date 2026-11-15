# Agent Deployment Safety

This project develops a practical method for understanding, assessing, and securing deployments of AI agents.

It focuses on serious, material harm: security incidents, privacy breaches, legal or regulatory violations, financial loss, operational disruption, and physical harm. Ordinary task failure matters here only when it can cross a control boundary and cause material consequences.

## How this project is being developed

The project is both a learning process and an engineering project. Concepts are discussed and understood before they are treated as settled or turned into software. We work incrementally and distinguish agreed decisions from proposals and open questions.

The object of assessment is a **specific deployment of an agent**, not a model in isolation. A safety conclusion applies to a defined combination of purpose, data, tools, permissions, inputs, autonomy, state, and operating environment.

## Documentation

- [Deployment parameters](docs/deployment-parameters.md) defines the seven pivotal properties used to analyze different agent deployments.
- [Internal research agent](docs/internal-research-agent.md) applies the model to a concrete deployment and compares a purpose-built Python implementation with a general-purpose harness represented by Claude Code.
- [Claude Code external security](docs/claude-code-external-security.md) defines the untrusted-sandbox and trusted-gateway architecture for containing the general-purpose harness.

## Current status

We have agreed on:

- The focus on material harm rather than general agent reliability
- The deployed system, rather than the model alone, as the unit of assessment
- Seven pivotal deployment parameters
- An internal research agent as the first concrete deployment
- Two implementation strategies: purpose-built Python and a general-purpose harness
- Claude Code as the sole concrete representative of the general-purpose-harness category for this example

The internal research agent's complete seven-parameter deployment profile and the high-level external security architecture for Claude Code are now documented. The Claude Code design treats the agent as an untrusted workload: it may compute broadly inside a disposable sandbox, while only narrow, trusted gateways can affect the outside world. No implementation has begun.
