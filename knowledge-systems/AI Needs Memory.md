# AI Needs Memory

AI needs memory.

Context windows provide short-term working memory.

But how do we get long-term memory?

There is no one definitive solution.

So, let's start by thinking through the problem.

## 1. How do we find knowledge?

Both humans and LLMs go through something like the following:

- First comes a need. To satisfy that need, we require knowledge.
- Through the use of superficial knowledge, we quickly analyze the need. This creates an initial spark of meaning, which often goes hand-in-hand with a bunch of words.
- Starting from that initial spark, we follow threads of knowledge until we reach the deeper knowledge that can satisfy the need in question.

Think about how we recall something.

We don't necessarily remember the exact information immediately. We start with a concept, a person, an event, or something associated with what we're looking for.

That leads us to another piece of information, which may lead to another.

We follow these associations until we reach the relevant knowledge.

In the case of AI, the big question is: **how do we build these threads of knowledge?**

## 2. What does the search process look like?

We can break the process down into three steps:

1. **Identify the query type.** Is the question asking for a fact, a relationship, an exhaustive list, a numerical calculation, or a broad synthesis? This determines the shape of our search.

2. **Locate relevant entities.** Use exact identifiers, keyword matching, semantic retrieval, or entity linking. Each of these may require a different retrieval mechanism or memory component.

3. **Traverse relevant relationships.** Starting from the entities located in step 2, follow their relationships to reach additional relevant knowledge.

Of course, this is a simplified model. Some questions can be answered directly, without traversing any relationships.

But for more complex questions, the connections between pieces of knowledge become increasingly important.

And step 3 is particularly interesting.

How do we represent these relationships so that an AI agent can discover and follow them efficiently?

## 3. Ontologies and knowledge graphs

One way to approach this is through an ontology and a knowledge graph.

I'm using these terms somewhat abstractly here, not necessarily referring to formal ontology languages or graph databases.

An **ontology** defines the concepts in a particular domain, what they mean, and how they can relate to one another.

A **knowledge graph** represents entities and their relationships, connecting different pieces of knowledge.

The ontology provides the conceptual structure. The knowledge graph represents the actual connections.

But we don't necessarily need specialized technologies to benefit from these ideas.

In fact, we can find something very close to them in ordinary information-management systems.

## 4. A filesystem is already a kind of knowledge structure

Consider a carefully organized filesystem.

It might have:

- Folder names chosen carefully.
- README files in many folders, defining what resides inside each folder and the folders downstream from it.
- References between files.
- Logs of operations.
- File indexes that help locate important information.

Actually, this is a pretty neat solution for personal data, and it works quite well by itself with Claude Code or Codex.

But consider what this structure already represents.

**The folder hierarchy provides a taxonomy.**

Files are classified into categories and subcategories. Their location communicates something about what they contain.

**Folder names and READMEs imply a rough ontology.**

They define concepts, explain what information belongs where, and establish conventions about how different parts of the system should be interpreted.

These definitions aren't necessarily precise or machine-readable in a formal sense, but they still provide meaning.

**References and navigation paths imply a rough knowledge graph.**

Files and directories are connected through containment, references, and other associations.

An AI agent can follow these connections, read the accompanying descriptions, and discover additional relevant information.

Of course, a filesystem is not literally a formal ontology or a knowledge graph in the conventional technical sense. The relationships are generally less expressive, and much of their meaning needs to be inferred from the documentation and conventions.

But conceptually, the similarities are significant.

A well-organized filesystem already contains much of the basic structure that a formal knowledge system would attempt to represent more explicitly.

And LLM-based agents are particularly useful here because they can interpret the natural-language descriptions that give this informal structure meaning.

## 5. When do we need something more formal?

Now suppose the amount of information grows significantly.

Or suppose the information is spread across multiple systems: files, databases, emails, messaging platforms, and external services.

A client might appear in a project directory, an accounting system, and an email account.

Each system may use different identifiers, different names, and different ways of organizing the information.

An agent can still search these systems and attempt to connect everything.

But it may have to rediscover the same relationships repeatedly. It may also struggle to recognize that different records refer to the same entity.

This is where a more explicit semantic structure might become useful.

For example, we could introduce:

- **A shared vocabulary:** Common definitions of important concepts.
- **An entity registry:** Stable identities for important people, organizations, projects, and other recurring entities.
- **Explicit relationships:** Selected connections that are difficult or unreliable to discover from the existing structure.

These additions can make important information easier to identify and connect.

Depending on the complexity of the system, a formal ontology or a dedicated knowledge graph might eventually make sense as well.

But this introduces another problem.

## 6. More structure also means more maintenance

Every additional structure introduces some amount of work.

If we create an entity registry, who maintains it?

If we represent relationships explicitly, how do we keep them synchronized with the underlying information?

If we duplicate descriptions and facts that already exist in READMEs, indexes, or source files, which version becomes authoritative?

And does an agent really need all those relationships predefined, or can it discover most of them efficiently from the existing structure?

We shouldn't assume that more formal knowledge representation automatically produces a better system.

A large knowledge graph that is difficult to maintain might be less useful than a smaller, well-organized collection of files.

The objective isn't to represent everything as entities and relationships.

It's to make knowledge available to AI agents in a way that is reliable, efficient, and maintainable.

## 7. Assess before building

So perhaps the right starting point isn't to build a knowledge graph.

It's to examine the knowledge system we already have.

How is information currently organized?

What concepts and relationships are already represented, explicitly or implicitly?

How effectively can agents navigate the structure?

Where do they actually struggle?

And how much additional maintenance would a more formal representation introduce?

Only then can we determine whether a shared vocabulary, an entity registry, an ontology, a knowledge graph, or something else would improve the system.

To make this process repeatable, I'm developing a **[Knowledge System Assessment skill](skills/knowledge-system-assessment/SKILL.md)**.

The skill provides a structured approach to examining existing knowledge systems, understanding their current organization and maintenance burden, identifying genuine limitations, and evaluating possible improvements.

It is deliberately technology-neutral. It doesn't assume that an ontology, a knowledge graph, or any particular architecture is the right solution.

It also puts considerable emphasis on avoiding duplicated information and unnecessary maintenance.

The outcome might be a recommendation to introduce a more explicit semantic structure.

Or it might be that the existing system already works well enough.

Because the point isn't to build an ontology or a knowledge graph.

**The point is to help AI access and use knowledge effectively, without making the system unnecessarily complicated.**
