# Internal Research Agent

## Status

This is the first complete working deployment profile. All seven parameters are defined. The Claude Code version now uses an untrusted-sandbox architecture: broad local computation is allowed inside a disposable worker, while trusted gateways mediate every effect on external systems.

## Deployment definition

The deployment is an employee-facing agent that answers research questions by searching public web sources and company-internal information, then produces a cited draft report for human review.

Its intended boundaries are:

- Employees submit research questions through an internal interface.
- The agent searches public web content.
- It searches internal documents the requesting employee is authorized to access.
- A hosted language model synthesizes the retrieved information.
- It returns a cited report to the employee.
- Reports, prompts, tool calls, and operational logs may be stored.
- It does not modify source documents.
- It does not send external communications.
- In the harness implementation, it may execute code and CLI commands only inside a disposable sandbox with no direct external authority.
- It does not autonomously make business decisions.

The initial system boundary is:

```text
Employee
   |
   v
Internal application and agent harness
   |-- Hosted language model
   |-- Public web search
   |-- Internal document search
   |-- Report storage
   `-- Logs and telemetry
   |
   v
Draft report for human review
```

The assessment distinguishes intended behavior from enforced capability. Intending that the agent access only authorized documents is not itself a security control. The document-search system must technically enforce the requesting employee's authorization boundary.

## Two implementation strategies

We will analyze the same deployment through two implementation strategies.

### Purpose-built Python system

This strategy builds only the orchestration and capabilities required by the research agent.

```text
Employee interface
        |
        v
Authentication
        |
        v
Python research orchestrator
        |
        v
Independent policy layer
   |-- Internal retrieval broker
   |-- Public web-search broker
   |-- Hosted-model gateway
   |-- Report storage
   `-- Controlled audit logging
```

Potential advantages include:

- Exposing only the tools the research task requires
- Using typed, narrowly scoped tool arguments
- Propagating the employee's identity through document retrieval
- Controlling how much document content reaches the model
- Separating internal retrieval from public web access
- Controlling storage, retention, redaction, network access, and budgets
- Avoiding generic CLI, filesystem-write, browser-action, and code-execution capabilities

The organization is also responsible for correctly building and maintaining authentication, authorization, connector security, user isolation, secret management, logging, error handling, dependency security, testing, and incident response.

The custom strategy therefore requires more foundational engineering but can make security part of a narrow design rather than an additional containment layer around broad capabilities.

### General-purpose harness represented by Claude Code

This category uses an existing harness that already supplies an agent loop, built-in tools, sessions, integrations, and extension mechanisms. Claude Code is the only concrete harness used in this project for now. Product-specific findings should not automatically be generalized to every harness.

Claude Code currently provides built-in file, search, execution, and web capabilities, with extension mechanisms including MCP, hooks, and subagents. It also provides allow, ask, and deny permission rules; filesystem and network sandboxing for Bash subprocesses; and lifecycle hooks that can inspect or block tool use. See the official documentation for [features](https://code.claude.com/docs/en/features-overview), [permissions](https://code.claude.com/docs/en/permissions), [sandboxing](https://code.claude.com/docs/en/sandboxing), and [hooks](https://code.claude.com/docs/en/hooks-guide).

These product details were checked against the official documentation on 2026-08-14 and must be reverified when an actual deployment is designed because harness capabilities and defaults can change.

For the research-agent profile, the initial hardening direction is:

- Permit useful shell, code-execution, and temporary-file capabilities inside an isolated, disposable sandbox.
- Prevent the sandbox from accessing host files, shared persistent volumes, privileged sockets, cloud metadata, or unrelated processes.
- Deny direct network access and route approved external effects through trusted gateways.
- Keep all reusable credentials outside the sandbox.
- Prevent modification of source documents and organizational systems.
- Avoid direct browser actions when controlled search and extraction are sufficient.
- Avoid delegation and persistent memory unless a requirement justifies them.
- Expose internal information through a narrowly defined connector.
- Require that connector to enforce employee authorization independently.
- Separate user sessions and stored state.
- Put sensitive-data and policy checks outside the model.
- Audit tool calls independently of model reasoning.
- Pin, test, and reassess harness and configuration updates.

Claude Code permissions and hooks remain useful defense-in-depth controls, but the primary security boundary is the sandbox and the external gateways. The design assumes that Claude Code may attempt unexpected operations inside the sandbox.

The concrete harness implementation will use the Python Claude Agent SDK inside the employee-facing application. The SDK supplies the Claude Code agent loop, while the application supplies authentication, policy enforcement, narrow research tools, report handling, and user isolation. The interactive Claude Code CLI may still be useful for an early engineer-operated prototype, but it is not the production architecture being assessed.

The high-level architecture is:

```text
                       TRUSTED CONTROL PLANE

Employee -> authentication and application policy
                         |
          +--------------+---------------+
          |              |               |
     model gateway  internal-search  public-search
     injects key    broker           broker
          |              |               |
==========|==============|===============|==========
                  SANDBOX BOUNDARY
                         |
                Claude Agent SDK worker
                - broad local computation
                - temporary filesystem
                - no reusable credentials
                - no direct network
                - no persistent state
```

All filesystem mounts, environment variables, sockets, devices, process namespaces, cloud metadata, network paths, and persistent storage that could cross this boundary are part of the containment design. The external brokers treat the sandbox as an untrusted client and independently validate every request.

The external containment design is documented separately in [Claude Code External Security Architecture](claude-code-external-security.md).

### Comparison

| Concern | Purpose-built Python | General-purpose harness |
|---|---|---|
| Prototype speed | Slower | Faster |
| Initial functionality | Must be built | Mostly provided |
| Capability minimization | Narrow by construction | Unneeded capabilities must be disabled or contained |
| Tool schemas | Fully controlled | Constrained by built-in and extension interfaces |
| Data-flow visibility | Can be designed explicitly | Requires product and configuration inspection |
| Security boundaries | Designed into the architecture | Combination of built-in and external controls |
| Upgrade behavior | Changed under our control | Upstream changes require review and regression testing |
| Assurance burden | Smaller if the system remains narrow | More existing behavior and configuration to verify |
| Engineering burden | Foundational implementation | Integration, containment, and hardening |

The implementation strategy does not replace the seven deployment parameters. It describes **control ownership**: which controls we build, which the harness supplies, which we configure, and which must be imposed externally.

For each deployment parameter, the example will record:

1. The risk shared by both implementations.
2. How the purpose-built Python system could control it.
3. How the Claude Code implementation could control it and where important limitations remain.

## Parameter 1: data sensitivity

### Data inventory

Data sensitivity cannot be represented by one label for the entire agent. Each form and location of data must be considered.

| Data | Possible sensitivity |
|---|---|
| Employee identity and prompt | Internal, confidential, or personal |
| Public web content | Public but untrusted |
| Internal documents | Internal, confidential, restricted, or regulated |
| Retrieved excerpts | Inherit the source document's classification |
| Generated report | Inherits sensitivity from its sources and inferences |
| Search queries | May reveal confidential research interests or prompt content |
| Logs and model traces | May contain copies of prompts, documents, and outputs |
| Access tokens and credentials | Highly restricted and should not enter model context |
| Usage metadata | May reveal employee activity and organizational interests |

The relevant data path is:

```text
Employee prompt
      |
      +--> internal retrieval --> retrieved excerpts --+
      |                                                |
      +--> public search query --> public results ------+--> model context
                                                        |
                                                        v
                                                  generated report
                                                        |
                              +-------------------------+------------------+
                              |                         |                  |
                              v                         v                  v
                         employee output          report storage      logs/traces
```

### Classification model

The initial generic classification model is:

1. **Public** — approved for unrestricted disclosure.
2. **Internal** — intended for employees but not public release.
3. **Confidential** — disclosure could materially harm the organization, customers, or employees.
4. **Restricted** — highly sensitive or specially controlled information, such as credentials, regulated records, or particularly sensitive personal and business data.

A client may use different classification names. Their existing system should be mapped to this model rather than replaced.

Classification levels may need handling tags such as personal data, health information, financial information, customer-confidential information, privileged legal material, export-controlled information, and credentials. Two items at the same confidentiality level can have different legal or contractual requirements.

### Data-handling principles

**Transformation does not remove sensitivity.** Summaries, embeddings, excerpts, and combined reports inherit the relevant classification of their sources unless a trusted process explicitly approves declassification. The model must not make declassification decisions by itself.

**Employee access does not automatically imply agent access.** An employee's ability to read a document does not by itself authorize sending the document to a hosted model, storing it in agent state, copying it into logs, combining it with other information, or retaining it after the task.

An agent data-access decision should require the intersection of:

```text
Employee permission
    AND agent data policy
    AND approved purpose
    AND provider and storage authorization
```

### Agreed data scope

The initial deployment may process:

- Public information
- Internal information
- Ordinary Confidential information

It may not process Restricted information or specially regulated categories. Expanding that scope would be a material deployment change requiring new analysis and controls.

### What provider trust means

An enterprise search or LLM provider can be approved to process Internal or Confidential information. Approval depends on the exact product and agreement, including training use, retention, processing location, subprocessors, access controls, incident obligations, and any special handling requirements.

Provider trust is limited:

- Trusting a search provider to protect a query does not make the web pages it returns trustworthy.
- Trusting an LLM provider to protect prompts does not mean the model will always handle the information correctly.
- Zero retention reduces storage risk but does not mean the provider never processes the data.

The simple data rules are:

1. Use only providers approved for the data classification and purpose.
2. Send only the information necessary for the task.
3. Treat web content as untrusted even when the search provider is trusted.
4. Keep credentials inside trusted connectors or proxies, outside model-visible context.
5. Use narrowly scoped, short-lived credentials if an exception is unavoidable.
6. Keep Restricted and specially regulated data outside this initial deployment.

### How the two implementations enforce the rules

Both implementations must satisfy the same rules. They differ in where the controls are created.

| Requirement | Purpose-built Python | Claude Code harness |
|---|---|---|
| Approve external providers | A provider gateway permits only destinations approved for the data class and purpose. | The deployment uses an approved commercial account and places any additional provider routing policy outside the agent. |
| Minimize model input | The orchestrator selects and limits retrieved excerpts before calling the model. | A narrow retrieval tool returns limited excerpts; a hook may inspect or reduce tool input, but the retrieval service remains the stronger boundary. |
| Protect public search queries | A dedicated query builder removes unnecessary internal details before calling the search provider. | Prefer a custom public-search tool with the same policy. If built-in `WebSearch` is used, a `PreToolUse` hook can inspect, block, or modify its query. |
| Enforce internal-document access | The retrieval service checks the employee identity, agent data policy, and approved purpose. | A custom MCP or SDK tool calls that same retrieval service. Claude Code does not make the authorization decision. |
| Handle untrusted web content | Search results are marked as external data and cannot directly authorize another operation. | Search output remains untrusted. Claude Code's web protections are useful but do not replace the deployment's tool and network boundaries. |
| Protect credentials | Connectors or a proxy attach credentials after checking the requested operation. The model never receives the secret. | Credentials remain in an external connector or proxy rather than prompts, tool results, or a model-readable workspace. This follows Claude Code's recommended proxy pattern. |
| Control reports | The application assigns classification, preserves citations, and checks employee access before storage or delivery. | The surrounding application performs the same checks after receiving the draft. Generated text is not trusted to classify itself. |
| Control sessions and logs | The application explicitly chooses what to store, redact, and delete. | Claude Code session transcripts require explicit handling. The SDK writes transcripts locally by default, so the deployment needs ephemeral local storage or a governed session-storage design and retention policy. |

Claude Code sources: [SDK tool availability](https://code.claude.com/docs/en/agent-sdk/custom-tools), [SDK permissions](https://code.claude.com/docs/en/agent-sdk/permissions), [hooks](https://code.claude.com/docs/en/agent-sdk/hooks), [secure credential proxy pattern](https://code.claude.com/docs/en/agent-sdk/secure-deployment), and [session storage](https://code.claude.com/docs/en/agent-sdk/session-storage).

### Claude Code containment boundary

Claude Code may use broad local tools inside its disposable sandbox. Only two task-specific tools cross into research services:

```text
Claude Code sandbox
   |-- shell, code, and temporary files stay local
   |
   |-- internal_search --> authorized retrieval broker
   `-- public_search   --> controlled search broker
```

The model connection also passes through an approved model gateway, and the surrounding application receives and stores the final report. The sandbox cannot create additional network destinations or directly persist its own output.

Claude Code tool permissions, hooks, and strict connector configuration reduce accidental exposure, but they are not the primary containment boundary. If an unexpected local tool becomes available, operating-system isolation, missing credentials, network denial, and external broker validation still prevent it from acquiring external authority.

## Parameter 2: impact and recoverability

### Credible serious harms

Impact is the material consequence the deployed system can credibly cause. Recoverability asks whether that consequence can actually be undone.

The assessment does not invent consequences that the system has no capability to produce. The harness may execute commands inside its sandbox, but those commands cannot directly modify operational systems or send external messages. Its principal serious risks therefore concern information rather than destructive action.

| Possible harm | Material consequence | Recoverability |
|---|---|---|
| Confidential information reaches the wrong employee or provider | Security, privacy, contractual, or legal incident | Low: disclosed information cannot reliably be undisclosed |
| Internal information leaks through a search query, report, or log | External disclosure | Low |
| A compromised or incorrectly authorized connector exposes documents | Potentially large internal data breach | Low, although the connector credential can be revoked |
| A false or manipulated report influences an important decision | Financial, legal, strategic, or reputational harm | Partial: the report can be corrected, but resulting decisions may not be reversible |
| Excessive searches or model calls consume resources | Financial cost or service disruption | Usually high: the session can be stopped and limits imposed |
| Improper retention or reproduction violates an obligation | Legal, contractual, or licensing consequences | Partial or low |

A read-only external authority surface is not harmless. Unauthorized reading can become irreversible disclosure. Confining code execution and filesystem changes to an ephemeral sandbox, while denying external write, communication, transaction, and infrastructure authority, substantially reduces the credible paths to operational damage, financial transactions, external commitments, and physical harm.

Impact and recoverability are outcome-specific. The deployment should not receive one undifferentiated recoverability label.

### Agreed decision-use boundary

The research report may support decisions, but it must not be the sole basis for consequential employment, legal, financial, regulatory, security, or safety decisions. Material claims used in those contexts must be independently verified from cited sources.

This is partly an organizational control. The application can label reports, preserve sources, record provenance, and require acknowledgements, but it cannot technically guarantee how a person uses information after reading it. Deployment policy, training, review procedures, and accountability are therefore required alongside technical controls.

### Prevention, limitation, and recovery

**Prevention:**

- Enforce employee document permissions.
- Keep credentials outside model context.
- Restrict approved providers and network destinations.
- Prevent sensitive information from entering inappropriate search queries.
- Store reports under the correct access restrictions.
- Confine general-purpose computation to the disposable sandbox and narrowly mediate every external effect.

**Impact limitation:**

- Bind each task to one employee and one authorization context.
- Limit the number and size of documents retrieved.
- Limit search, model, time, and cost budgets.
- Isolate session state between employees.
- Exclude Restricted and specially regulated data.

**Recovery and response:**

- Stop active sessions and revoke connector credentials.
- Determine which documents were accessed and which destinations received data.
- Delete stored reports and transcripts where possible.
- Correct affected reports and identify important decisions that relied on them.
- Notify affected parties and initiate the applicable incident process.

For disclosure, prevention remains more important than deletion because removing a stored copy does not prove that every recipient has forgotten or destroyed the information.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Remove high-impact external actions | Do not implement handlers for shell, file modification, messaging, transactions, or infrastructure. | Permit local computation but give the sandbox no direct external write, communication, transaction, or infrastructure path. |
| Bound information access | Pass the employee identity and task policy to narrowly scoped retrieval services. | Custom SDK or MCP research tools call the same external retrieval services; Claude does not decide authorization. |
| Prevent unrestricted egress | Route outbound calls through explicit search and model gateways. | Give the SDK worker no generic egress; only the trusted model and research gateway interfaces cross the sandbox boundary. |
| Limit task scale | Enforce document, query, token, time, concurrency, and cost limits in application code. | Apply the same application limits and also set SDK turn and budget limits. External limits remain authoritative. |
| Control report use | The application labels the draft, preserves citations, restricts access, and records review. | The surrounding application applies the same controls to the SDK result before storage or display. |
| Stop harmful execution | Cancel the task worker and revoke its short-lived access. | Cancel the SDK process or isolated worker and revoke the same external access. |
| Investigate an incident | Use application audit events to reconstruct retrieval, provider, and report flows. | Combine application events with governed SDK tool-call and session records, without treating raw transcripts as unrestricted logs. |

The important safety boundary is external in both implementations: even if the agent loop behaves incorrectly, tool availability, retrieval authorization, network policy, credentials, and application limits constrain the material consequences it can produce.

## Parameter 3: tool capability

### Agreed capability boundary

The two implementations use different internal capabilities but expose the same two boundary-crossing research activities:

```text
internal_search
public_search
```

The surrounding application receives, validates, and stores the final draft. The model does not receive a report-storage tool.

### Local capabilities inside the Claude Code sandbox

Claude Code may use:

- Shell commands
- Python and other approved local programs
- Temporary files and directories
- Local document-processing and data-analysis tools
- Local intermediate artifacts

These capabilities operate only on task-scoped data inside a disposable environment. The sandbox has no host or employee home-directory mounts, reusable credentials, privileged sockets, cloud metadata, direct external network, or shared persistent state.

### `internal_search`

This tool:

- Searches only approved internal information sources.
- Uses employee identity and authorization context supplied by the application, not by model-generated arguments.
- Returns limited excerpts, source metadata, classification metadata, and citations.
- Cannot modify source documents.
- Cannot retrieve documents outside the employee's effective authorization boundary.
- Does not expose connector credentials to the model.

The model may supply research terms and permitted filters. It may not select a different identity, credential, tenant, backend service, or authorization policy.

### `public_search`

This tool:

- Sends a minimized query to an approved search provider.
- Returns extracted passages, source URLs, titles, and citations.
- Cannot submit forms, authenticate to websites, download files, or perform browser actions.
- Cannot access private network addresses or internal services.
- Treats all returned content as untrusted data.
- Applies server-side limits to query length, result count, response size, rate, and cost.

The initial deployment does not permit general browsing or arbitrary URL fetching. Controlled search and extraction can be reconsidered only if evidence shows that it is insufficient for the research purpose.

### Boundary-crossing capabilities deliberately absent

The sandbox has no direct capability for:

- Arbitrary network requests or URL fetching
- Host, employee, or shared filesystem access
- Interactive actions on external websites
- Email or messaging
- Database or source-document writes
- Cloud, infrastructure, or identity administration
- Credential retrieval
- Persistent memory
- Subagents or delegation
- Scheduling

These external effects must be technically unavailable, not merely prohibited by the system prompt. A local tool such as `curl` or `kubectl` may exist inside the sandbox, but without network access, credentials, host mounts, or privileged sockets it has no corresponding external authority.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Expose only two external research activities | Register only the two narrow Python tool handlers in the custom agent loop. | Retain useful local tools, but expose only the two custom SDK or MCP tools across the sandbox boundary. |
| Constrain tool arguments | Define narrow schemas and validate every field before dispatch. | Define the same narrow schemas in custom tools; enforce important limits again in the external service. |
| Prevent identity selection | Bind authenticated employee context outside the model and omit identity and credential fields from tool schemas. | Bind the same context in the surrounding application or retrieval service; do not accept model-supplied identity. |
| Prevent arbitrary network access | The application can call only the configured model, retrieval, and search services. | Default-deny the sandbox network and expose model, retrieval, and search through trusted gateways rather than generic HTTP access. |
| Contain unexpected tools | Unknown tool names fail in the custom dispatcher. | Assume unexpected local tools may run; isolate their filesystem, processes, credentials, sockets, and network while also using Claude Code permissions as defense in depth. |
| Audit use | Record validated calls and results at the application boundary with appropriate redaction. | Record the same application events and use SDK tool events as supporting evidence rather than the sole audit boundary. |

An `allowed_tools` list alone is not the Claude Agent SDK security boundary because it can approve listed tools without removing every unlisted tool. The primary security boundary is the sandbox plus the external capability gateways.

The security effect of any tool is evaluated as:

```text
tool + environment + credentials + connectivity
```

## Parameter 4: privilege and scope

### Agreed authorization model

The deployment uses delegated per-user authorization. Every internal retrieval request is evaluated using the requesting employee's real permissions, while the employee's credential remains outside model context.

```text
Employee authenticates
        |
        v
Application creates trusted authorization context
        |
        v
Agent requests internal_search(query)
        |
        v
Retrieval service combines:
   employee permissions
   + agent data policy
   + approved purpose
   + data-class restrictions
   + request limits
        |
        v
Authorized excerpts
```

The model supplies research terms and permitted filters. It cannot select or modify employee identity, tenant, authorization token, data-class allowance, backend service, retention policy, or server-side result limits.

### Scope of one session

Each research session is restricted to:

- One authenticated employee
- One organization or tenant
- Documents that employee is authorized to access
- Public, Internal, and ordinary Confidential data
- Search and limited excerpt retrieval
- Server-bounded result count and response size
- A limited task duration and resource budget
- Approved model, search, and retrieval destinations
- Output returned through the employee application

The public-search credential authorizes search only, has server-side rate and spending limits, remains inside the search connector, and is never exposed to the model.

The agent worker has no unrelated filesystem access, user home-directory access, unrestricted network access, or ambient cloud credentials.

### Why a broad service account is not the default

A shared service account with organization-wide document access could turn the agent into a privilege bridge. One authorization or filtering failure could expose information far beyond the initiating employee's normal access.

If a source system cannot support delegated authorization, a broad service identity may be used only behind an independent retrieval broker that performs equivalent user-level authorization and is tested as a critical security boundary. This is an exception, not the preferred architecture.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Bind employee authority | The application passes authenticated user context directly to the retrieval handler outside model-generated arguments. | The application or external MCP service binds the same context; the model sees only the query schema. |
| Enforce document permissions | The retrieval broker evaluates source ACLs and agent data policy before returning excerpts. | The same external broker remains authoritative; prompts, hooks, and SDK permissions do not replace document authorization. |
| Limit model authority | The custom dispatcher exposes no identity, credential, tenant, or destination parameters. | Custom tool schemas omit those fields, and only the exact approved tools are available. |
| Limit worker privilege | Run the application under a dedicated identity with access only to the required services. | Run each SDK worker in an isolated, minimally privileged environment with no ambient credentials or unrelated mounts. |
| Bound public search | Use a dedicated search credential and enforce rate, cost, and response limits in the connector. | Use the same external connector; the SDK worker does not possess a reusable provider credential. |
| Prevent cross-user state | Key every task, cache, report, and audit event to the authenticated employee and enforce access in storage. | Apply the same application controls and isolate SDK working directories and session transcripts between employees. |

Privileges are enforced by identity systems, retrieval services, network controls, and storage boundaries outside the model. The agent may request an allowed operation but cannot grant itself more authority.

## Parameter 5: input exposure and trust

### Trust is not one property

For each input, the assessment asks three separate questions:

1. Is its source authentic?
2. Does it have authority to instruct the agent?
3. Are its factual claims reliable enough for the intended use?

An authentic internal document can be authorized for retrieval while having no authority to change the agent's security rules. It may also contain outdated or incorrect claims.

### Agreed trust hierarchy

| Input | Permitted authority |
|---|---|
| Application security policy | Defines mandatory security boundaries |
| Authenticated employee request | Defines the research goal within those boundaries |
| Internal documents | Evidence to retrieve and summarize; not agent instructions |
| Public web content | Untrusted evidence; not agent instructions |
| Tool errors and external messages | Operational data; not instructions |
| Model output | A proposed draft subject to application controls and human review |

All retrieved content, including authenticated internal documents, has no instruction authority. It may describe company procedures as evidence, but it cannot authorize tools, expand permissions, select a different provider, reveal credentials, change retention, or override deployment policy.

### Prompt injection and enforcement

Separating instructions from retrieved data in prompts and structured messages helps the model reason correctly, but it is not a complete security boundary. Natural-language content can still manipulate model behavior.

The enforceable protection comes from the surrounding system:

- Only the two approved research activities can cross the sandbox boundary.
- Shell, code, and file operations are confined to temporary task state inside the sandbox.
- Tool schemas exclude identity, credential, tenant, and destination selection.
- Retrieval services independently enforce authorization.
- Provider and network destinations are fixed outside the model.
- Credentials remain outside model context.
- Query, result, time, and cost limits are server-enforced.
- No direct external messaging, write, browser-action, or transaction capability exists.
- The application controls report storage and access.
- Important report claims require citations and human verification.

If retrieved content manipulates the model, these controls still limit the consequences available to the manipulated agent.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Control authoritative instructions | Construct system and task messages explicitly in application code. | Supply an explicit application system prompt and tightly control which Claude Code settings, project instructions, skills, and plugins can load. |
| Separate task intent from policy | Treat the employee request as a task input that cannot alter hard-coded policy or tool enforcement. | Apply the same separation; SDK instructions do not override external tool, identity, or network controls. |
| Mark retrieved material as data | Return structured results containing content, source, classification, and trust metadata. | Custom SDK or MCP tools return the same structured results rather than injecting retrieved text as system instructions. |
| Preserve provenance | Carry source identifiers and citations through synthesis and report validation. | The surrounding application validates that SDK output citations correspond to retrieved sources. |
| Limit injected instructions | Do not execute or forward actions found inside retrieved text. | Allow local computation but rely on sandbox containment and external gateways to prevent injected text from creating unauthorized outside effects. |
| Control configuration inputs | Load prompts and policies only from application-owned configuration. | Isolate the SDK worker and explicitly restrict filesystem settings and connector configuration rather than inheriting arbitrary user or project configuration. |

The system does not attempt to prove that all prompt injection has been detected. It limits what a successfully manipulated model can cause.

## Parameter 6: autonomy and mediation

### Activity-level autonomy

Autonomy is assigned to each activity, not once to the entire agent.

| Activity | Authority and mediation |
|---|---|
| Start a research task | The employee initiates it. |
| Choose and refine search terms | The agent may do this within policy and resource limits. |
| Choose internal or public search | The agent may choose between the two approved tools. |
| Execute a bounded search | Automatic after deterministic authorization, data, query, and budget checks. |
| Retry a search | Automatic within turn, time, retry, and cost limits. |
| Change identity, tenant, provider, permissions, or data class | Not permitted. |
| Hold temporary task state | Controlled by the application for the active task. |
| Save the completed report | Requires an explicit employee decision. |
| Share or publish a report | No agent capability exists. |
| Use a material claim in a consequential decision | Requires independent human verification from cited sources. |
| Continue later or run on a schedule | Not permitted in the initial deployment. |

The flow is:

```text
Employee starts task
        |
        v
Agent researches autonomously
        |
        v
Every tool call passes deterministic controls
        |
        v
Application validates and labels draft
        |
        v
Employee reviews
        |
        v
Employee decides whether to save or use it
```

The employee does not approve every search call. Frequent approval prompts would add friction and encourage mechanical approval. Human involvement is concentrated at meaningful boundaries: initiating the task, persisting or sharing the result, and relying on it for consequential decisions.

Full reports and transcripts are not retained automatically. Temporary state may exist while the task runs, and limited audit metadata may be retained under policy. The employee explicitly chooses whether to save the final report in an approved location.

### General-purpose reasoning with task-specific authority

The purpose-built system defines the two research activities directly. The general-purpose Claude Code harness retains broad local computation inside a disposable sandbox, but exposes the same two narrow activities across the security boundary.

```text
Purpose-built Python:
known activities -> specific controls -> bounded autonomy

Claude Agent SDK:
broad local computation -> sandbox boundary -> narrow external capabilities
```

The reasoning engine remains flexible. Its authority does not.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Require employee initiation | The application creates a task only from an authenticated request. | The application invokes a fresh SDK task only from the same authenticated request. |
| Mediate searches | The custom dispatcher calls policy-protected search services. | Custom SDK tools call the same services; SDK permission checks provide an additional layer. |
| Bound iteration | Application code limits turns, retries, duration, tokens, and cost. | Apply the same external limits and configure SDK turn and budget limits. |
| Prevent authority changes | Identity, providers, policies, and tool registry are immutable to model output. | Keep these outside model arguments and prevent the SDK worker from loading or enabling alternatives. |
| Control persistence | The application holds temporary state and saves a report only after employee action. | The application does the same and separately controls the SDK's local session files. |
| Prevent communication and action | No corresponding handlers exist. | The sandbox has no direct network, credentials, or external gateways for communication and action. |

The activity profile, rather than the choice of agent loop, defines the permitted autonomy.

## Parameter 7: persistence and propagation

### Agreed persistence profile

| Component | Behavior |
|---|---|
| Active task state | Temporary; expires when the task finishes, is cancelled, or times out |
| Agent conversation transcript | Not retained automatically after the task |
| Audit metadata | Retained according to policy with minimal sensitive content |
| Final report | Saved only after an explicit employee decision |
| Saved-report permissions | Enforced by the approved document store |
| Cross-session agent memory | None |
| Shared memory between employees | None |
| Scheduled or recurring tasks | None |
| Subagents and delegation | None |
| Retries | Limited to the active task by count, time, and budget |
| Credentials | Connector-held or short-lived; not persisted in task state |
| Automatic reuse of reports | None |

### Saved reports are governed documents, not agent memory

When an employee saves a report, it becomes an ordinary governed company document. It retains its classification, citations, provenance, ownership, access controls, and applicable retention policy.

A saved report does not automatically:

- Enter agent memory
- Become a future research source
- Become trusted because the agent generated it
- Become visible to other employees
- Trigger another workflow or agent

If an organization later wants saved reports to become searchable sources, a separate ingestion process must evaluate classification, authorization, provenance, quality, retention, and whether the report is appropriate for reuse.

This prevents a mistaken or manipulated output from silently becoming authoritative context and amplifying through future work.

### Task termination and expiry

Each task has fixed limits for duration, turns, retries, token use, search calls, and cost. When the task completes, times out, is cancelled, or violates a limit:

- Further tool calls stop.
- Task-bound authorization expires.
- Temporary worker and session state is deleted.
- Minimal audit metadata records the termination reason.
- No task resumes without a new authenticated employee request.

### Control placement in the two implementations

| Control objective | Purpose-built Python | Python Claude Agent SDK |
|---|---|---|
| Keep task state temporary | Hold task context in memory or an isolated ephemeral worker. | Give each SDK task an isolated temporary configuration and working directory and destroy it afterward. |
| Prevent automatic session reuse | Do not implement cross-session agent memory or automatic resume. | Do not use SDK continue or resume behavior; explicitly manage the local transcripts the SDK creates. |
| Control report persistence | The application saves only an employee-approved report to the governed document store. | The same surrounding application saves the result; the model has no storage tool. |
| Retain safe audit evidence | Store structured event metadata with redaction and a defined retention period. | Store the same application events; do not use unrestricted raw SDK transcripts as the default audit store. |
| Bound retries and duration | Enforce task deadlines, retry counts, and budgets in the orchestrator. | Enforce the same external limits and configure SDK turn and budget limits as defense in depth. |
| Prevent propagation | Provide no memory, scheduling, delegation, messaging, or automatic ingestion mechanisms. | Remove those capabilities from the SDK tool and configuration surface. |
| Revoke authority | Expire task context and connector access when the task terminates. | Terminate the isolated SDK worker and expire the same external access. |

The Python Claude Agent SDK writes session data locally during operation, so deletion and isolation must be designed explicitly. Application-controlled storage remains the authoritative location for any state that is intentionally retained.
