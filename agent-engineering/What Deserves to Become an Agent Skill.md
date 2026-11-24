# What Deserves to Become an Agent Skill?

OpenAI frames skills as reusable instructions and supporting resources for repeatable tasks. They are particularly useful when good results depend on a repeatable approach, multiple steps, structured outputs, policies, standards, or other specialized context that the model would not reliably infer on its own.

That does not mean every recurring task should become a skill. A skill is useful when this added structure materially improves how the agent performs the work.

The key question is:

> **What does the skill add that the base agent would not already do reliably?**

---

# 1. What makes a good skill candidate?

Skills are most useful when good execution depends on one or more of the following:

- a repeatable process;
- multiple steps that should happen in a particular way;
- organizational policies or standards;
- specialized procedures or context the model would not reliably infer;
- specific templates, schemas, or output formats;
- consistent use of tools or sources;
- quality checks that should happen every time.

A useful mental model is:

> **A task describes what needs to be done.**
> **A skill captures how the work should be done.**

For example:

**Weak skill candidate**

> Analyze these sales results.

The base model can probably handle this.

**Stronger skill candidate**

> Analyze sales results using our review framework, compare against plan and prior quarter, apply our materiality thresholds, classify deviations using our taxonomy, and produce our standard management-review format.

The second example contains reusable procedure and context the model would not otherwise reliably know or apply.

---

# 2. When a skill may not make sense

A skill may be unnecessary when:

- the task is repeatable, but the base agent already handles it well;
- every case requires a substantially different approach;
- the instructions merely restate generic good practice;
- the underlying process is still unclear or disputed;
- the skill would remove useful flexibility;
- maintaining the skill would cost more than the improvement it provides.

For example:

> Summarize these three articles.

Even if this task happens every day, a skill may add little if the base agent already performs it reliably.

Compare that with:

> Review these articles using our monitoring framework, ignore defined low-signal events, classify developments into our categories, and escalate anything meeting our risk criteria.

Both tasks repeat. Only the second clearly contains reusable procedure that may justify a skill.

---

# 3. Evaluate against the baseline

A skill should be evaluated by comparing the **same agent with and without the skill**.

Use a representative set of normal, difficult, and edge cases. Keep the agent, tools, inputs, and tasks as comparable as possible.

Before testing, define what the skill is supposed to improve. For example:

> This skill should reduce missed compliance checks and make risk classification more consistent.

This determines what you should measure. Relevant dimensions may include:

| Dimension            | What to evaluate                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------- |
| **Correctness**      | Whether the skill improves factual, analytical, or procedural accuracy.                       |
| **Completeness**     | Whether required steps, checks, or output elements are less likely to be missed.              |
| **Method adherence** | Whether the agent follows the intended procedure rather than substituting a generic approach. |
| **Consistency**      | Whether comparable cases are handled in a reliably similar way.                               |
| **User effort**      | Whether users need less prompting, explanation, correction, or rework.                        |

Additionally, assess:

| Dimension            | What to evaluate                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------- |
| **Flexibility** | Whether the agent can still adapt when a case does not fit the normal process.        |
| **Efficiency**  | Whether the improvement justifies any additional complexity, latency, or maintenance. |



Look at regressions as well as improvements. A skill may improve consistency while reducing flexibility, for example.
