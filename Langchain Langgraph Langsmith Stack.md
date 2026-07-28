# LangChain, LangGraph, and LangSmith: A Ground-Up Breakdown of the AI Engineering Stack

## Introduction

LangChain, LangGraph, and LangSmith are three names that keep appearing in every serious AI engineering discussion today, not in research papers, but in production systems that retrieve from real data, call external tools, run multi-step reasoning loops, and get monitored like actual software.

Most people who have heard of them, or even used them, have only a fuzzy picture of what each one does, why all three exist, and how they fit together. This documentation provides a ground-up breakdown of the entire stack: what each piece solves, how they relate, and how they combine into a coherent architecture for building AI systems that go well beyond a single API call.

## The Problem with a Simple LLM API Call

When you hit an LLM through an API, the interaction is simple: you send a prompt, and the model returns text. That is the whole interaction.

Consider a simple document Q&A system. A user asks a question about an internal policy document. Before you even call the model, you need to:

- Find the right document out of potentially thousands.
- Construct a prompt that includes the relevant context.
- Have the model generate an answer.
- Format that answer or pass it to another step.

> **Key insight:** What you have now isn't an LLM call, it's a pipeline. You're retrieving data, constructing prompts dynamically, calling the model, parsing outputs, and chaining steps together.

If you build all of this from scratch, you end up writing the same boilerplate over and over again. That is the problem **LangChain** was built to solve.

## LangChain: The Vocabulary of AI Applications

LangChain gives you the building blocks to structure applications around language models rather than just treating them as isolated API calls. Its key abstractions each solve a specific pain point.

### Prompt Templates

Instead of hard-coding prompts as static strings, you define reusable templates that inject context at runtime. For example: "Explain the concept of [topic] in simple terms," where the application substitutes the topic based on what the user is asking.

This is a simple idea, but it means your prompts are structured, testable, and maintainable rather than scattered as string literals across your codebase.

### Chains

A chain is a sequence of steps where the output of one component becomes the input of the next. For example: summarize a document, extract key insights, generate a user-facing response.

Each step involves the model, but the steps are wired together into a single cohesive workflow. LangChain handles the plumbing so you don't have to.

### Tools

By default, an LLM can only generate text. It can't run a calculation, check today's stock price, or query your database. Tools fix that.

> **Definition:** A tool is a function the model can invoke when it decides it needs external information.

The model says, "I need to run a web search." The framework runs it, gets a result, and passes that back into the model's context. Now the model can continue reasoning with fresh, real-world data. This is the shift that turns a text generator into something that acts.

### RAG (Retrieval-Augmented Generation)

LLMs have a hard limitation: they only know what they were trained on. You can't retrain a model every time your company updates its internal documents. RAG solves this elegantly.

Instead of cramming knowledge into the model:

1. You store your documents in a vector database.
2. When a user asks a question, the system retrieves the most relevant chunks using semantic similarity search.
3. The retrieved chunks are injected into the prompt.
4. The model answers based on that context.

> **Key point:** The model doesn't need to know everything. It just needs access to the right information at the right time.

## The OAuth and Authentication Challenge

A problem that doesn't show up until you try to ship is authentication. Your LangChain agent can call tools, but the moment that agent needs to act on behalf of a real user, read their Gmail, post to Slack, update a Jira ticket, push a GitHub issue, you need OAuth, and not once, but for every single provider your agent touches.

When your application kicks off an OAuth flow, that flow needs to handle three things that are your responsibility by default:

- The OAuth handshake itself.
- Storing and managing the tokens it gets back.
- Refreshing those tokens before they expire mid-task.

All of this produces a **connected account**, a live, authenticated link between one of your users and one external provider. That connected account then authenticates against the actual OAuth providers (Gmail, Slack, Jira, GitHub), each with its own token logic and scopes.

Multiply this by every provider your agent needs and every user in your system, and the complexity compounds. Your LangChain tools request a token, and you have to return tokens at tool call time via an encrypted token vault, so the tokens live outside the model's context window, otherwise there is a huge surface area for attacks and vulnerabilities.

### Scalekit (Sponsor)

This is what Scalekit offers out of the box. When your agent calls a tool, Scalekit handles:

- Authentication and OAuth token refresh.
- Credential isolation.
- Access rule enforcement, including delegated identity per agent, scoping, and RBAC.
- Executing the tool call against whatever system you need, SaaS API, MCP server, or database.
- Logging every action: who, what, when, and which agent performed it.

You focus on the agent and business logic while this unified gateway handles the complex authentication work. Scalekit supports 3,000+ tools including Gmail, Slack, Jira, Salesforce, Notion, GitHub, and more. It also supports custom OAuth app credentials, production branding, and higher rate limits.

## From Chains to Agents

Once you have tools and chains working, developers naturally start pushing further: what if, instead of a fixed pipeline, the model could decide what to do next? That is an **agent**.

An agent doesn't just answer one question. It reasons about a problem, picks an action, executes it, observes the result, and then decides what to do next, repeating this until the task is complete.

### The ReAct Loop (Reasoning and Acting)

The loop works as follows:

1. **Analyze** the task, what do I need to do?
2. **Choose an action**, which tool should I invoke?
3. **Execute** it, run the tool.
4. **Observe** the result, what did I learn? Is the task done?
5. **Repeat** if not done.

Here is where the engineering challenge hits: these workflows are no longer linear. They have loops. They branch. They call different tools depending on what they find. They have state information that needs to persist across multiple reasoning steps.

> **Key point:** You're no longer writing a script. You are engineering a dynamic, stateful system. Simple chains can't handle this cleanly, which is exactly why LangGraph exists.

## LangGraph: The Control Flow Layer

LangGraph takes a fundamentally different approach to structuring AI systems. Instead of chains, it uses **graphs**.

A LangGraph application is made up of:

- **Nodes**: Each node is a unit of work, an LLM call, a tool execution, a retrieval step, or a custom function.
- **Edges**: Each edge defines how the system moves between nodes, and these transitions can be conditional.

### Example: A Research Agent

Consider a research agent that:

1. Gets a question.
2. Runs a web search.
3. Reads the result.
4. Decides whether it has enough information to answer.
5. If not, it searches again with a refined query.

This is a loop, not a straight line, and a chain can't model it cleanly. In LangGraph, that loop is explicit, the reasoning step is a node, the search is a node, and the "do I have enough" decision is an edge condition. The system loops back or exits based on state.

### Shared State

Rather than passing data manually between steps, LangGraph maintains a **shared state object** that the entire graph reads from and writes to. Each node picks up the current state, does its work, updates the state, and passes control forward.

> **Key point:** This makes the system auditable. You know exactly what information each step had, what it changed, and what triggered the next move.

## LangSmith: The Visibility Layer

A problem that doesn't surface until you actually try to ship one of these systems: when something goes wrong, and it will, you need to figure out why.

- Was it the prompt?
- Did the retrieval step pull the wrong documents?
- Did the agent choose the wrong tool?
- Did the model hallucinate?

In a system with multiple reasoning steps, tool calls, and retrieval pipelines, the failure could be anywhere. And unlike traditional software where failures are crashes, the failure mode here is often a subtly wrong answer.

LangSmith fills this observability gap.

### Tracing

LangSmith records every step: the input prompt, the model's response, which tools were called, what they returned, the retrieved documents, and intermediate reasoning, all visualized as a structured trace. When something produces a wrong answer, you open the trace and see exactly where things went sideways.

### Evaluation

Observability isn't just about debugging, it's also how you get better over time. AI outputs aren't binary; a response can be partially correct, off topic, or subtly misleading.

LangSmith lets you build evaluation data sets and run your system against them automatically, measuring:

- Accuracy
- Relevance
- Latency
- Token usage

So when you change a prompt or tweak your retrieval logic, you're not guessing whether it helped, you can measure it.

### Prompt Experimentation

Small wording changes can dramatically shift model behavior. LangSmith lets you run A/B comparisons across prompt variants against the same data set, making your iteration systematic.

### Production Monitoring

Once you are live in production, LangSmith tracks latency, error rates, token usage, and tool invocation patterns under real traffic, similar to New Relic or Grafana for distributed systems.

## The End-to-End Architecture

Here is what the complete flow looks like:

1. A user sends a request.
2. **LangGraph** determines the execution path, which steps to run, in what order, with what branching logic at each node.
3. **LangChain** components do the actual work, prompt construction, retrieval, the model call, and tool execution.
4. **LangSmith** records the entire thing, every input, every output, every decision point, every tool invocation.

> **Summary definition:** LangChain is the vocabulary. LangGraph is the control flow. LangSmith is the visibility layer. Each one solves a distinct problem, and together they cover the full surface area of building AI systems that actually work in production.

## Criticisms and Limitations

None of this means LangChain is perfect. Common criticisms from engineers include:

- **Heavy abstractions:** When something breaks, you're often debugging through multiple layers of framework code before you even get to your actual logic. For a simple one-step LLM call, the overhead generally isn't worth it, just hit the API directly.
- **Fast-moving API changes:** There has been pain around how fast the library evolved. Teams that built on early versions found themselves refactoring when the API shifted under them.

LangGraph was actually born out of some of this frustration. Instead of rigid chains, you get explicit graphs where the control flow is visible. LangSmith exists because observability was an afterthought in the original design, when your pipeline is a black box, you can't improve what you can't see.

### When to Use This Stack

- If you are prototyping something small, you probably don't need this stack.
- Where it earns its complexity is multi-step pipelines, dynamic agents, and systems you need to evaluate and iterate on over time. That is where these three pieces start pulling their weight together.

## Closing Thoughts

A year ago, building with AI meant writing clever prompts. Today, it means designing systems with retrieval pipelines, reasoning loops, tool orchestration, and observability baked in. That shift is already happening.

The engineers building the next generation of AI products aren't just prompt crafting, they are architecting. This stack is one of the core foundations they are building on.
