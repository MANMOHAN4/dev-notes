# Building an AI Agent with Real Memory Using Oracle AI Database 26AI

## Introduction

You build an AI agent. It works perfectly in a demo , you ask it to help a user manage their task, and it responds correctly. But when that same user comes back the next day and asks, "Hey, remember the task I added yesterday?", the agent has no idea what they are talking about.

> **Key point:** This isn't because the code broke or the LLM failed. It's because the agent genuinely has no memory of anything that happened before this exact moment.

This documentation walks through building an agent from scratch that actually remembers, and explains why the standard fix most people use is the wrong architectural decision. The implementation runs on Oracle AI Database 26AI.

## Why Agents Forget

This comes down to how LLMs actually work. Every time you call an LLM, you are sending it a fresh prompt. There is no internal state and no running memory between calls. The model processes what you give it and responds , that's it.

When an agent finishes one conversation and starts a new one, it starts completely clean. It has no idea what happened before unless you explicitly tell it.

> **Key point:** This is by design. LLMs are stateless, which makes them predictable and scalable. The problem isn't the LLM , the problem is what we do (or don't do) about it.

## The Common (but Flawed) Fix: Three Separate Memory Systems

Most developers solve this by adding a memory layer built from three separate systems:

1. **Short-term store**: An in-memory cache that holds recent conversation context within a single session.
2. **Long-term store**: A relational database for facts that need to persist across sessions , user preferences, history, and anything else that needs to persist.
3. **Vector database**: A dedicated store for embeddings and semantic search, so the agent can find related context even when a question uses different words than what was stored.

> **The problem:** Three systems means three connection strings and three sets of credentials to manage. Every time the agent needs context, it has to make calls across all three and assemble the result before it can even begin to answer. This works, but it's expensive, fragile, and creates a coordination problem that gets worse as the agent becomes more complex.

## A Better Way: Memory Next to the Data

The real problem is that business data , the stuff an agent is actually supposed to reason about , already lives in a database, but the agent's memory lives in three completely separate systems. Every meaningful action requires a round trip across the business data and across three different memory stores.

> **Key question:** What if memory lived right next to the data? What if your database could handle all three types of memory at once?

### Oracle AI Database 26AI: A Converged Database

Oracle AI Database 26AI is a **converged database** , it handles relational data, JSON documents, graph relationships, and vector embeddings all in the same engine, all queryable in a single statement.

> **Key point:** Instead of calling three separate systems for short-term context, long-term facts, and semantic search, the agent makes one call , one connection string, one system that stays consistent by default.

## Setting Up Oracle AI Database 26AI Locally

This setup can run for free on a laptop using the Oracle 26AI Free Edition Docker image, and takes about 2 minutes.

### Setup Steps

1. Go to `container-registry.oracle.com` and create a free Oracle account (takes about a minute).
2. Navigate to the database section and find the free repository , this is the Oracle AI Database 26AI Free image, which is open source with no license acceptance needed.
3. Open a terminal. The free image is public, so it can be pulled directly without logging in. The image is about 3.5 GB, so allow a few minutes for the download.
4. Run the container, setting `ORACLE_PWD` as the password for the database admin user (pick anything secure).
5. Follow the logs and wait for the line "Database is ready to use," which takes about 60 seconds. Press Control+C when it appears.
6. Set up a Python virtual environment and install the dependencies.
7. Add an OpenAI API key to the shell environment so it persists across sessions.

> **Tip:** Oracle now runs locally with no cloud account and no credit card required. If the container ever stops, simply run `docker start oracle26ai` to bring it back.

## Building the Agent: Two Versions

With Oracle running locally and dependencies installed, the next step is building the agent. Two Python files are used for comparison:

1. **agent_without_memory.py**: A standard LangChain agent , no memory, no Oracle, just an LLM with a tool.
2. **agent_with_memory.py**: The same agent and LLM, but with Oracle wired in as the memory backend.

## Demonstration: The Broken Version (No Memory)

### Session 1

The agent is introduced to information: a name, and the fact that the person is a software engineer and founder of Bitemark. The agent responds, confirming it knows who the person is.

### Session 2

A new session is started with the same agent and LLM , just a new call. When asked "What do you know about me?", the agent says it has no information about the person.

> **Key point:** One new call and everything is gone. The agent didn't break , the code ran fine. There was simply nowhere to put the memory, so there was no memory.

## Demonstration: The Version with Oracle Memory

### Session 1

The same introduction is given, with the same LLM. This looks identical to the broken version so far.

### Session 2

A new call is made, passing zero history into the invocation , nothing. The agent still knows exactly who the person is.

> **Key point:** That information is not coming from a variable passed in , it's coming from Oracle. The agent loaded the conversation history before generating a single token.

### Session 3: Semantic Recall

In Session 1, the description used was "software engineer and founder of Bitemark." In Session 3, the question asked is "What kind of work do I do?" , using completely different words, without saying "engineer" or "Bitemark."

> **Key point:** The agent still gets it right, because Oracle didn't just store the text , it stored the meaning. The vector embedding sits right next to the relational data in the same engine. One query, no separate system to call, no sync job to run. That is what a converged database means in practice: you're not choosing between remembering facts and understanding meaning , you get both in one place.

## How It Works in Code

### The Stateless Version (No Memory)

This implementation uses **LangChain** as the orchestration layer. LangChain provides a standard way to connect an LLM to tools and external systems without writing all the plumbing manually , essentially, the glue between agent logic and everything it needs to interact with.

Key components:

- **The prompt**: Where the agent gets its instructions , a system message, user input, and a scratchpad for the agent to reason through tool calls.
- **create_openai_functions_agent**: Wires the LLM and the prompt together.
- **AgentExecutor**: Runs the agent and returns the final output.

> **Key point:** This setup is clean, simple, and completely stateless , and that stateless nature is the problem.

### The Version That Remembers

This version uses the same LangChain structure, with additions made before the agent runs:

1. **Oracle connection**: One connection string , the same database the application data already lives in.
2. **Vector store**: `OracleVS`, LangChain's Oracle vector store integration. This is configured by passing the connection, the embedding model, the table name, and the distance strategy. Oracle creates a table and manages the vector index automatically.

> **Note on flexibility:** OpenAI embeddings are used in this example, but the setup works with any LLM , LangChain supports Anthropic, Gemini, or a local model. The Oracle memory layer does not care which one is chosen.

3. **Conversation history table**: A plain SQL table with session ID, role, content, and timestamp fields.

### The Three Memory Functions

These three functions are the heart of the memory system:

- **save_memory**: Writes every message to the history table. If the message is from the user, it also embeds the text and stores the vector in Oracle. One function, two writes, one database.
- **load_memory**: Fetches recent conversation history for a session, giving the agent short-term context.
- **search_memory**: Runs semantic similarity search across all stored embeddings. This is what made Session 3 work , different words, same meaning, and Oracle found the connection.

> **Key point:** Three functions, one Oracle database , no separate cache, no separate vector store.

### Putting It Together: run_agent

The `run_agent` function brings everything together:

1. Before every response, the system loads recent history and searches semantically for relevant context from any past session.
2. Both the recent history and the semantic search results are injected into the system prompt before the LLM sees the user input.
3. After the response, the exchange is saved back to Oracle.

> **Key point:** Every call is informed by everything before it, and every response gets stored for future calls. One database handles all three memory jobs.

## Availability and Contribution

All of this code is available in the Oracle AI Developer Hub on GitHub. Developers interested in building on this can fork the repository, add their own use cases, and submit a pull request. Oracle's developer team actively reviews community contributions.

## Closing Thoughts

Agent memory isn't a hard problem , it's an architecture problem. Understanding where data and memory should live, and choosing a system that unifies them, resolves much of the complexity that typically comes with building agents that remember.
