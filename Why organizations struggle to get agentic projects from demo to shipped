# Why organizations struggle to get agentic projects from demo to shipped

Shipping agentic systems is full of complexities.

## Operations

**TL;DR: You cannot automate a process you do not understand. Most organizations rely on processes full of undocumented judgment, exceptions, and human coordination that nobody has ever had to formalize.**

Before building an agent, you need to understand the operation it is supposed to perform.

* What is the goal?
* What decisions need to be made?
* What information is required?
* What actions are allowed?
* What happens when something goes wrong?

Many organizations cannot answer these questions precisely because humans have been filling the gaps.

People rely on judgment, undocumented knowledge, workarounds, and informal coordination.

An agent forces those assumptions into the open.

And once the process is understood, it often becomes obvious that simply automating the existing workflow is not enough. The workflow itself may need to change.

That can mean:

* redesigning processes,
* changing responsibilities,
* changing approval flows,
* restructuring teams,
* building new supporting systems.

The difficult part is often not automating the operation.

It is figuring out what the operation should be.

## Evaluation

**TL;DR: If you cannot measure whether the system succeeds, you cannot systematically improve it. This is difficult because most organizations lack representative eval sets, explicit success criteria, and established ways to measure probabilistic, multi-step systems.**

Agent development without evaluation quickly becomes anecdotal:

* "This prompt seems better."
* "This model felt smarter."
* "It worked when I tried it."

That is not enough.

Teams need:

* representative tasks,
* explicit success criteria,
* known failure cases,
* regression tests,
* production metrics.

And agents have to be evaluated at several levels:

* Did the model make the right decision?
* Did it take the right actions?
* Did the workflow complete correctly?
* Did the user get the desired outcome?
* Was that outcome worth the cost and risk?

Evaluation is not something added after development.

It is the mechanism by which development becomes engineering.

## Data

**TL;DR: Agents expose data problems that humans quietly work around. Most organizations have data systems designed to support slow-moving people, not fast-moving autonomous systems that need clean, explicit, machine-readable information at every step.**

Agents make decisions based on information.

If that information is missing, stale, ambiguous, inconsistent, or inaccessible, the agent will make worse decisions.

Humans compensate for bad data constantly. They:

* recognize that a field is wrong,
* know which document is outdated,
* remember an exception,
* ask a colleague.

Agents do not have that background knowledge unless the system provides it.

At low volume, these problems look like minor annoyances.

At machine scale, they become systematic failure modes.

Agentic systems therefore tend to reveal the true quality of an organization's data.

## AI

**TL;DR: The hard AI problem is usually getting the right information and decisions to happen at the right time. This is difficult because many of the engineering patterns for context, retrieval, memory, tool use, and orchestration are still immature and highly task-dependent.**

Even after the surrounding system exists, building the agent itself is not trivial.

The central problem is context.

For each decision:

* What does the agent need to know?
* Where does that information live?
* How do we find it?
* How do we decide what is relevant?
* How do we present it to the model?
* What should the model decide itself?
* What should remain deterministic?

Many teams collapse this into "use RAG" or "put the data in a vector database."

That is often insufficient.

Real systems may need combinations of:

* structured queries,
* semantic retrieval,
* metadata,
* graphs,
* reranking,
* memory,
* tool calls,
* task-specific logic.

The underlying problem is not retrieval alone.

It is building an information system that gives the model the right context at the right moment.

We are still learning the standard patterns for doing that well.

## Security

**TL;DR: The more an agent can do, the more carefully you have to constrain what it can access and affect. This is difficult because useful agents need real authority, while most security systems were designed around predictable software and human users—not autonomous software deciding what to do next.**

Agentic systems become useful when they can access data and take actions.

That is also when they become dangerous.

The basic questions are simple:

* Will the agent break our system?
* Will it break somebody else's system?
* Will it leak our data?
* What should it be allowed to access?
* Which users, systems, tools, and sources do we trust?
* What happens when the agent consumes malicious or misleading information?
* How do we limit the damage when something goes wrong?

The core problem is authority.

An agent should have enough authority to do useful work, but no more than necessary.

That makes permissions, isolation, authentication, authorization, audit logs, and sandboxing part of the core architecture.

## Costs

**TL;DR: You often do not know the economics until the system behaves like the real product. This is difficult because agent cost emerges from many interacting variables that prototypes rarely reproduce accurately.**

Agent cost is not just model cost.

It depends on:

* context size,
* model choice,
* number of steps,
* tool calls,
* retries,
* retrieval,
* infrastructure,
* human escalation,
* failure rates.

Those variables interact.

A cheaper model that fails more often may produce a more expensive system.

A longer workflow may improve quality while destroying unit economics.

And prototype behavior may not resemble production behavior.

So cost is often difficult to estimate before the system exists.

The relevant question is ultimately simple:

> What does it cost to successfully complete one unit of useful work?

## Ownership

**TL;DR: A system that spans many disciplines needs someone who owns the whole outcome. This is difficult because most organizations divide responsibility across teams, while agentic systems cut directly across those boundaries.**

Agentic systems cut across:

* operations,
* data,
* infrastructure,
* security,
* AI,
* integrations,
* product,
* economics.

That creates an organizational problem.

Who is responsible:

* when the model changes?
* when an API breaks?
* when costs double?
* when security finds a vulnerability?
* when the business process changes?
* when quality slowly degrades?

A prototype can survive through individual effort.

A production system needs durable ownership.

Many projects stall because the technology crosses organizational boundaries more easily than responsibility does.

## Reliability

**TL;DR: It is not enough for an agent to succeed once; it has to produce the right outcome consistently. This is difficult because probabilistic errors compound across multi-step workflows, and most organizations are not used to designing processes around inherently variable components.**

LLMs are probabilistic.

The same system can receive similar inputs and behave differently.

An agent may:

* misunderstand an instruction,
* retrieve the wrong information,
* choose the wrong tool,
* make the wrong decision,
* take the wrong action.

These probabilities compound.

An agent that makes the right decision 95% of the time may look impressive in isolation. But a workflow requiring many such decisions can have a much lower end-to-end success rate.

Improving reliability therefore requires:

* constraining decisions where possible,
* validating important outputs,
* checking intermediate results,
* reducing unnecessary agentic steps,
* escalating uncertain cases,
* using deterministic logic where variability provides no value.

The relevant measure is not whether the agent *can* complete the task.

It is how often the entire system produces an acceptable outcome.

## Integration

**TL;DR: An agent is only useful if it can interact with the systems where real operations happen. This is difficult because enterprise systems are fragmented, inconsistent, and rarely designed to be safely orchestrated by autonomous software.**

Most useful actions do not happen inside the model.

They happen in:

* CRMs,
* databases,
* email,
* internal tools,
* document systems,
* payment systems,
* APIs,
* other infrastructure.

Agents therefore need integrations.

Those integrations are rarely clean.

Different systems have different:

* identities,
* permission models,
* APIs,
* data formats,
* operational assumptions.

Once an agent starts coordinating across them, the problem begins to look like ordinary distributed systems engineering.

The AI may be new.

The integration problem is not.

## Robustness

**TL;DR: Even a reliable agent operates inside an unreliable world. This is difficult because production systems must survive malformed inputs, broken integrations, partial failures, changing dependencies, and unexpected situations that demos can simply ignore.**

Reliability is about producing the right result.

Robustness is about what happens when things go wrong.

An agentic system still has to deal with ordinary software failures:

* APIs go down,
* requests time out,
* data arrives malformed,
* permissions change,
* rate limits are hit,
* dependencies change,
* actions succeed only partially,
* state becomes inconsistent,
* users do unexpected things.

Agentic systems add another source of failure on top: the agent itself may take an unexpected path through the system.

Production systems therefore need:

* retries,
* fallbacks,
* timeouts,
* state recovery,
* idempotency,
* failure isolation,
* monitoring,
* graceful degradation.

None of this is uniquely AI engineering.

It is serious software engineering.

The difficulty is that many agentic projects begin as lightweight prototypes where these problems are invisible. Moving to production means building all of the engineering that the demo was able to skip.

## Summary

Shipping requires:

* the process has to be understood,
* quality has to be measurable,
* the data has to be usable,
* the AI system has to be designed properly,
* the agent has to be constrained,
* the economics have to make sense,
* somebody has to own the outcome,
* the agent has to be reliable,
* the integrations have to work,
* the system has to be robust.
