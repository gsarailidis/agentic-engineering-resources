# Claude Code External Security Architecture

## Purpose and scope

This document defines the high-level external security architecture for using the Python Claude Agent SDK in the internal research-agent deployment.

It does not attempt to make Claude Code itself a trusted security boundary. The design assumes that the agent may be mistaken, manipulated, or compromised and may attempt unexpected operations.

The objective is:

> Even if Claude Code behaves maliciously, it cannot cross the sandbox boundary except through a small number of independently controlled capabilities.

The central design principle is:

> Broad computation inside the sandbox; narrow authority across the sandbox boundary.

## High-level architecture

```text
                       TRUSTED CONTROL PLANE

Employee -> authentication -> task controller
                                 |
          +----------------------+----------------------+
          |                      |                      |
     model gateway        internal-search broker  public-search broker
          |                      |                      |
          +----------------------+----------------------+
                                 |
========================= SECURITY BOUNDARY =========================
                                 |
                    per-task Claude Code sandbox
                    - broad local tools
                    - temporary task data
                    - no real credentials
                    - no generic network
                    - no shared storage
                    - destroyed after the task
```

Only four kinds of information or effects may cross the boundary:

1. Model inference through the model gateway
2. Authorized internal retrieval through the internal-search broker
3. Controlled public retrieval through the public-search broker
4. The final draft returned to the initiating application

Everything else is denied.

## 1. Sandbox isolation

Each employee task receives a separate, disposable sandbox. Claude Code may use shell commands, local code execution, document-processing programs, data-analysis libraries, and temporary files inside that sandbox.

The sandbox has no:

- Host filesystem or employee home-directory mounts
- Shared writable storage
- Container-management or orchestration sockets
- Cloud-instance metadata
- Host environment variables
- Access to other sandbox processes
- Devices or privileged operating-system capabilities
- Reusable external credentials
- Generic outbound network access

The sandbox also has limits on CPU, memory, disk, process count, runtime, model spending, and tool calls.

Because the agent may execute arbitrary local commands while processing untrusted content, an ordinary shared-kernel container should not be assumed to provide the strongest boundary for Confidential information. A stronger container boundary such as gVisor or a per-task microVM is the preferred direction. The exact technology has not yet been selected.

Anthropic's official guidance discusses containers, gVisor, and virtual machines as isolation choices for Agent SDK deployments: [Securely deploying AI agents](https://code.claude.com/docs/en/agent-sdk/secure-deployment).

## 2. No ambient authority

The sandbox contains no credential that works directly against a real external system.

It receives only a temporary task identity that:

- Identifies one task
- Is bound to one authenticated employee
- Expires quickly
- Is accepted only by the trusted gateways
- Cannot be used directly with document, model, search, cloud, or other providers

The gateways hold and use the real credentials.

```text
Claude requests internal_search
              |
              v
Broker validates trusted task context
              |
              v
Broker applies employee authorization and data policy
              |
              v
Broker retrieves permitted information
```

Identity, tenant, permissions, and credential selection come from the trusted control plane, not from model-generated tool arguments.

## 3. No generic external network

The sandbox does not receive ordinary internet access or a generic HTTP-request capability. Approved effects are expressed as application-level operations:

```text
model_inference(...)
internal_search(...)
public_search(...)
return_draft(...)
```

It cannot request:

```text
http_request(arbitrary_url, arbitrary_body)
```

A domain allowlist alone is not treated as a complete security boundary. An allowed service may accept uploads, relay data, redirect traffic, or obscure encrypted request contents. Anthropic also documents limitations of network filtering without TLS inspection in its [sandboxing guidance](https://code.claude.com/docs/en/sandboxing).

Application-level gateways make the permitted external effects explicit and validate their meaning rather than merely their destination.

## 4. Trusted capability gateways

Every gateway treats the sandbox as an untrusted client. It independently validates identity, operation, parameters, data policy, scope, rate, and cost.

### Model gateway

The model gateway:

- Uses an approved enterprise model account
- Holds the real model credential
- Enforces allowed model operations and model selection
- Applies task, token, rate, and cost limits
- Implements the approved provider data and retention policy

### Internal-search broker

The internal-search broker:

- Gets employee identity from the trusted task controller
- Enforces the employee's document permissions
- Enforces the agent's permitted data classifications and purpose
- Limits document count and excerpt size
- Records source and authorization metadata
- Returns no reusable credentials

Claude cannot change identity, tenant, permission, data class, or backend by changing its tool arguments.

### Public-search broker

The public-search broker:

- Uses only an approved enterprise search provider
- Offers controlled search and extraction, not arbitrary URL fetching
- Limits query length, result count, response size, rate, and cost
- Prevents access to internal and private network destinations
- Treats returned material as untrusted content

The provider must be approved for the information that may appear in search queries. If the provider is not approved for Confidential information, query filtering alone is not a complete defense because a manipulated model may encode information in unexpected ways. A stricter deployment would separate public research from confidential internal research so the public-searching process never receives the confidential content.

### Output boundary

The final draft:

- Returns only to the initiating employee's application
- Preserves classification, citations, and provenance
- Is not automatically shared or published
- Is not automatically saved
- Is not automatically added to agent memory or future research sources

## 5. Task termination and revocation

When a task completes, fails, times out, violates a limit, or is cancelled:

- The sandbox stops and is destroyed.
- Its temporary filesystem disappears.
- Its task identity expires.
- Gateways reject further calls.
- No session resumes automatically.
- Minimal audit metadata records the termination reason.

Destroying the complete environment is preferred to trying to clean an agent-controlled filesystem.

## 6. External monitoring and response

The authoritative audit trail comes from the trusted task controller and gateways rather than from logs produced by Claude Code.

It records, subject to data-minimization requirements:

- Employee and task identity
- Gateway operation requested
- Authorization outcome
- Data sources accessed
- Provider destination
- Resource consumption
- Result classification
- Termination reason

The control plane can:

- Revoke a task immediately
- Stop a sandbox
- Disable an individual gateway
- Disable the complete deployment
- Identify affected documents and external destinations

## What this architecture does not solve

External containment limits what the untrusted agent can cause outside its sandbox. It does not guarantee:

- That a report is correct
- That every citation supports the associated claim
- That an authorized employee will use the report appropriately
- That approved providers never process the information sent to them
- That every legal, contractual, or organizational requirement has been satisfied

Those concerns require the provider, data-governance, report-verification, human-review, and organizational controls defined in the broader deployment profile.

## Current decisions and open choices

Agreed:

- Claude Code is treated as an untrusted workload.
- Broad local computation is allowed inside a disposable sandbox.
- The sandbox contains no reusable external credentials.
- Generic external network access is denied.
- External effects pass through narrow, trusted gateways.
- Gateways independently enforce identity, authorization, scope, data policy, and budgets.
- Workers and task authority expire at task termination.

Not yet selected:

- The concrete sandbox technology
- The concrete model and search providers
- Gateway protocols and implementation technology
- Exact quotas and time limits
- Audit-retention periods
- Whether public and confidential research need separate processing phases
