# Pivotal Deployment Parameters

## Purpose

This document defines a small set of properties that determine the major security requirements of an agent deployment. The parameters are intended to apply across different agent types and implementation technologies.

The object of assessment is a **specific deployment of an agent**, not the model in isolation. The same model can present very different risks depending on its tools, permissions, data, inputs, operating environment, and level of autonomy.

A safety conclusion therefore applies to a defined deployment configuration. Material changes to that configuration may require reassessment.

## The seven parameters

### 1. Data sensitivity

**Question:** What information can the agent receive, retrieve, infer, store, or disclose?

Data sensitivity determines requirements for access control, isolation, encryption, retention, logging, residency, and disclosure. The full data path must be considered:

```text
Input -> model context -> tool call -> memory -> logs -> output -> external destination
```

Relevant data includes:

- Data supplied directly by a user
- Data retrieved through tools
- Data inferred or combined by the agent
- Persistent memory and state
- Logs, traces, and telemetry
- Tool arguments
- Outputs and external transmissions

### 2. Impact and recoverability

**Question:** What material harm could the deployment cause, and can the resulting change be reversed?

This parameter covers security, privacy, legal, financial, operational, and physical consequences. It also covers the criticality of affected systems, the scale of possible harm, and whether recovery is practical. It determines the need for approval, staging, backups, rollback, recovery procedures, or complete prohibition of particular actions.

### 3. Tool capability

**Question:** What kinds of effects can the available tools produce?

Tool capability determines requirements for sandboxing, allowlists, parameter validation, network restrictions, and action mediation. Tools should be evaluated by their possible effects rather than by product name alone.

The security effect of a tool depends on its environment:

```text
tool + environment + credentials + connectivity
```

A general-purpose harness may retain broad local computation inside a disposable sandbox while having only narrow, mediated ways to affect the outside world. Local capability and external authority must be assessed separately.

### 4. Privilege and scope

**Question:** What does the agent's identity actually authorize, and how broadly?

This includes accessible resources, allowed operations, environments, tenants, accounts, and the number of objects that one action can affect. It determines requirements for least privilege, identity isolation, environment separation, tenant boundaries, and blast-radius limits.

Tool capability and privilege are distinct:

```text
Tool capability x actual permission x permitted scope
```

A tool describes the kind of operation that is possible. The identity and its permissions determine what a particular deployment can actually affect.

### 5. Input exposure and trust

**Question:** Who or what can influence the agent's decisions?

Influences may include direct users, web content, email, documents, tickets, retrieved records, tool results, and other agents. This parameter determines requirements for authentication, trust boundaries, untrusted-content handling, prompt-injection defenses, and separation between instructions and data.

### 6. Autonomy and mediation

**Question:** Which decisions and actions can occur without independent approval?

**Core principle: autonomy is a property of an activity, not of an agent.** A single deployment may autonomously perform bounded retrieval while requiring human approval to persist, communicate, or act on the result. A label such as "high-autonomy agent" hides these important differences.

For every meaningful activity, the assessment asks:

1. Who initiates it?
2. Who selects its parameters?
3. What deterministic controls apply?
4. Is human approval required?
5. Who or what executes it?
6. What limits its scope and repetition?
7. Can its effects be reversed?

The result is an **autonomy profile**, not one autonomy score for the whole deployment.

Useful activity-level mediation patterns include:

1. The agent recommends an action but cannot prepare or execute it.
2. The agent prepares an action for review.
3. The agent executes after human approval.
4. The agent executes automatically after independent, deterministic policy checks.
5. The agent selects and executes the action without independent mediation.

This parameter determines requirements for human approval, deterministic authorization, transaction staging, separation of duties, and action-level policy enforcement.

For deployments that use a general-purpose model or harness, the preferred design is:

> The reasoning engine may be general-purpose; the authority surface should be task-specific.

A purpose-built system defines a small vocabulary of meaningful activities directly. A general-purpose harness often begins with broad technical capabilities from which the model can assemble new activities. It may retain broad computation inside an isolated sandbox, but authority that crosses the sandbox boundary should be narrow, consequence-aware, and independently mediated.

### 7. Persistence and propagation

**Question:** Can the agent retain state, operate later or repeatedly, or pass authority and information to other actors?

This includes memory, scheduling, retries, long-running tasks, recurring operation, delegation, and multi-agent workflows. It determines requirements for authorization expiry, reapproval, rate and cost limits, state integrity, provenance, monitoring, and emergency shutdown.

Persistence asks what survives the current task. Propagation asks whether information, authority, or errors can spread into later sessions, other users, other agents, or organizational systems.

For every persistent or propagating element, the assessment asks:

- What is retained, where, and for how long?
- Who can read, change, or delete it?
- Does it automatically re-enter future model context?
- Is its original authorization still valid when it is reused?
- Can incorrect or manipulated output become authoritative state?
- Can tasks retry, resume, or execute later without renewed approval?
- Can information or authority pass to other agents or systems?
- How are execution, credentials, and retained state revoked?

Persistent agent output should not automatically become trusted future input. Memory, report ingestion, delegation, and scheduled execution require explicit purpose, provenance, authorization, expiry, and containment.

## How the parameters combine

The parameters can be divided into two groups.

**What harm is available:**

- Data sensitivity
- Impact and recoverability
- Tool capability
- Privilege and scope

**How easily that harm can be activated or amplified:**

- Input exposure and trust
- Autonomy and mediation
- Persistence and propagation

A deployment becomes particularly concerning when it combines sensitive or critical assets, consequential tools, broad privileges, untrusted influence, autonomous execution, and persistent or scalable operation.

This is not yet a numerical risk formula. It is a structured way to identify the deployment properties that materially change the required security architecture.

## Initial tool capability classes

The initial method covers common tool classes rather than maintaining a product-specific catalog.

1. **Web and browser access** introduces untrusted content and possible outbound data flows. Search-only retrieval and an authenticated browser capable of taking actions are materially different capabilities.
2. **File, document, and knowledge retrieval** can expose confidential information or cross authorization boundaries. This includes filesystems, document stores, source repositories, and retrieval systems.
3. **Database and business-system APIs** can read or modify durable organizational state. This includes databases, CRM systems, ticketing systems, and other SaaS applications.
4. **Communication tools** can send email, messages, tickets, notifications, or public content, potentially disclosing information or representing an organization externally.
5. **Code execution and CLI access** can provide filesystem, process, network, credential, and installed-tool access. Full CLI access is a general-purpose capability rather than a narrowly bounded tool.
6. **Source-control and software-delivery tools** can modify code, open or merge changes, run CI, create releases, or initiate deployments.
7. **Cloud and infrastructure controls** can modify services, compute, containers, networks, storage, secrets, IAM, and production resources.

Specialized classes such as financial transactions, medical systems, or industrial controls can be introduced when a deployment requires them.

Every tool should ultimately be characterized by questions such as:

- Can it read data?
- Can it write persistent state?
- Can it execute code or commands?
- Can it communicate externally?
- Can it access sensitive information?
- Can it affect production?
- How broad is its scope?
- Are its effects reversible?
- Can its parameters be constrained independently of the model?

## How parameters determine controls

```text
Data sensitivity       -> data-access and disclosure controls
Impact/recoverability  -> approval, staging, backup, and rollback
Tool capability        -> sandboxing and tool-level restrictions
Privilege/scope        -> least privilege and blast-radius limits
Input exposure         -> trust boundaries and injection defenses
Autonomy               -> independent action mediation
Persistence            -> expiration, budgets, monitoring, and shutdown
```

These parameters form the current high-level model. They will be refined only when analysis of concrete deployments shows that something important is missing, overlapping, or insufficiently precise.
