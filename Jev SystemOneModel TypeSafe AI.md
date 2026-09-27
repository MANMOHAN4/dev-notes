# Understanding Jef: Typesafe AI's System 1 Model

## Introduction

This documentation explains what **Jef** actually is, what problem statement it solves, and why it is not just another LLM model. There are already many LLM models in the market — GPT 5.5, GPT-6 Astra, Claude's Opus 5, Opus 5.5, and various other models. Jef is fundamentally different.

Terms like "type-safe AI," "System 1 models," and "Jef" have been circulating recently. This documentation demystifies what Jef exactly is and why it represents a fundamental change — a model built from the ground up that is highly optimized and innovative.

> **Note:** This is based on an article published by Typesafe AI, released on September 15, 2026, introducing System 1 models and Jef.

## Understanding the Problem Statement First

Before diving into what System 1 models are, it helps to understand the underlying problem through a simple system design example — setting AI aside for a moment.

### A Typical System Design

Consider a standard system design:

1. A **user** talks to a **reverse proxy** (e.g., Nginx).
2. The reverse proxy routes requests to a set of **horizontally scaled servers** (e.g., an auto-scaling group on AWS).
3. These API services accept requests, process them, and return responses back to the user.

Real systems are rarely this simple. These servers typically also push events to something like **Kafka**, which has **consumer groups** processing those events (for simplicity, consider two consumers). These consumers make rapid decisions and perform database operations against systems like **MongoDB** and **PostgreSQL** — some CRUD operations here, some reads and writes there.

> **Key point:** This is roughly what a high-level system design typically looks like.

### Natural Language Becomes the New Interface

With AI, natural language has become the primary interface. Before AI/LLM models existed, everything was static — systems worked on structured data formats like JSON and XML. A user's request would arrive in JSON format, get processed, trigger events, and get stored in a database like MongoDB.

As LLMs emerged, people started moving away from rigid JSON toward natural language: "What if I can control my entire system using natural language? What if everything is automated?" For example, a user could simply ask, "What is my account balance?" in plain English.

### How LLMs Get Injected Into the Architecture

When such a natural language query comes in, it's sent to the backend, where instead of a plain database operation, there are LLM models everywhere. For example:

1. The query goes to an LLM backend server (e.g., OpenAI).
2. The LLM has access to tools (e.g., a "fetch balance" tool).
3. The LLM calls this tool, potentially in a looping mechanism.
4. It gets back a response as JSON and sends it back to the user.

> **Key point:** This is how modern systems have evolved — LLMs have been injected at nearly every layer, whether for tagging, categorizing a user query, or incorporating human-in-the-loop feedback. This is a significant layer that has been added on top of modern systems.

## Identifying the Bottleneck

Consider another example to identify the fundamental problem when trying to scale this architecture in production.

### Example: A Refund Request

Suppose a user says, "I need a refund for my order number 37." What actually needs to happen here is simple:

1. Add some kind of **urgency tag** to this query.
2. Extract the **order ID**.
3. Decide what needs to be processed next (a yes/no decision).

This kind of query, when passed to an LLM through a system prompt with specific questions to answer, gets processed via tool calls and may return a structured JSON output — or it may not.

> **Key insight:** This is exactly where the problem starts. If all that's needed is a predefined set of answers to predefined questions, using a traditional, general-purpose LLM for this is actually overkill.

### Why Traditional LLMs Are Overkill for This

Traditional LLMs are trained to **generate** text — they are generative in nature (GPT literally stands for "Generative Pre-trained Transformer"). This means if a query says "write a poem" or "write a YouTube script" or "write code to do something," these are genuinely generative tasks well-suited to models like OpenAI's, Claude's, or Grok's models — because these models are trained and heavy enough to handle such tasks.

But consider tasks like:

- "How urgent is this message?" (on a scale of 0 to 5) — this is not generation, it's **categorization/labeling**.
- "Is my order shipped?" — this requires only a **true/false** answer based on a database lookup, not generation.
- "Categorize this article" with labels like "AI/ML" or "Backend" — again, not generation.

> **Key point:** For these kinds of decision-making tasks — where there are predefined outputs and what's really needed is simple NLP-based intelligence — using a full generative model wastes more time and more tokens, because these models are inherently heavy and generative by nature. This can also lead to unpredictable output formatting — a true/false question might come back as "Yes," "OK," "True," "true," or "Alright, yes, this order is shipped," because the model's training data biases it toward generating conversational text.

## Introducing System 1 Models and Jef

For tasks that require decision-making with predefined possible outputs — essentially NLP-based intelligence rather than generation — a **System 1 model** can be used instead of a traditional generative LLM.

> **Definition:** Typesafe AI is the company working on System 1 models, designed specifically to solve this class of problem. Their first released model is called **Jef**.

According to Typesafe AI's documentation: "Jef is Typesafe's flagship model for the first System 1 model. Send state and typed questions, get structured answers you can code using directly."

### How Jef Works: Example

Consider this example message: *"Hi, I have been trying to connect my Stripe account for three days and the integration keeps failing. I am losing sales. Please help me as soon as possible."*

With a traditional approach, you'd ask an LLM a question like: **"Does this message express urgency?"** (a yes/no question). Jef returns a strict **boolean** — yes or no — without elaborating with commentary like "Have you tried this, have you tried that?" the way a context-aware model like OpenAI or Claude might, since those models have broader context about topics like Stripe.

> **Key point:** Jef only answers based on the given instruction — nothing more. These models are trained from the ground up using reinforcement learning for calibrated decisions, making them extremely fast since they carry no unnecessary context (e.g., no knowledge about Stripe specifically) — they simply analyze what the user said and determine urgency based on that alone.

### Example Use Cases

Jef supports several structured query types:

- **Choice-based questions**: "What programming language is this code written in?" with options like Python, JavaScript, TypeScript, Go, and Rust — Jef simply detects and returns the matching option, without analyzing the code for bugs or offering commentary.
- **Categorization**: "What type of meeting is this, based on the title and description?" — returns one of the given options.
- **Routing decisions**: "My running shoes arrived late and in the wrong size. Can I swap them for size 10? Which team should handle this — exchanges, delivery, or charges?" — Jef returns just one of the given options, without generating any explanatory text.
- **Scoring**: "The export button crashes the settings page in Safari. It works in Chrome, but a few of our customers use Safari." — given specific criteria, Jef returns just a severity score, without additional commentary or suggested fixes.
- **Boolean-style ("Now/No-type") questions**: "I asked three times now — can I please just talk to a real person?" → Is this customer asking for a human agent? Yes/No. Has the customer contacted support before? Yes/No — again, no extra commentary, just a direct answer to what was asked.

> **Key point:** This is exactly why Jef is extremely fast — it only answers what is asked, with no elaboration.

## Performance: Speed and Cost Comparison

A benchmark comparison shows Jef against a traditional LLM answering the same structured question set. The traditional generative model has to generate its full response, introducing latency and a risk of hallucination as it produces free-form text. Jef, by contrast, returns its answers almost instantly.

According to Typesafe AI: "We have built our first public model, Jef, available today. Jef achieves a similar level of intelligence on System 1 tasks compared to existing LLMs, while being two orders of magnitude faster and more efficient."

> **Key point:** Jef is faster because it isn't trained to generate open-ended text — it simply answers pinpointed questions — and it is also significantly cheaper to run.

## When to Use Jef

Jef makes sense whenever a system needs:

- Categorization of text.
- Yes/no decisions.
- Determining what should be done next from a predefined set of options.
- Any kind of processing based on text that does not require generating new content.

### Example Use Case: An LLM Router

One particularly strong use case is an **LLM router**. When a user asks a question, Jef can determine which downstream model is best suited to handle it — for example, routing coding questions to Claude, generic questions to GPT, and reasoning-heavy questions to Grok.

> **Key point:** This kind of LLM routing or decision-tree-based upstream selection is exactly where Jef fits well — sitting in the middle of an "open router"-style architecture and providing fast routing decisions.

## Closing Thoughts

Jef can be thought of as the "if-else" logic of the LLM world — it is fast and simply gets the job done for structured decision tasks. While generative models like GPT and Claude can technically perform these same tasks, using them for simple categorization or yes/no decisions is overkill. Jef, being pretrained from the ground up specifically for these System 1 tasks, offers a far more efficient alternative wherever a system needs a quick, predictable, pinpointed answer rather than open-ended text generation.
