# Harness Engineering for AI Agents: Explained with the Car Analogy

## Introduction

"Harness" is a term that has become a hot topic recently — you may have seen it on Twitter, LinkedIn, or in job postings mentioning "We are hiring harness engineers" or "We want engineers who can write harnesses for agents." This term became especially popular after OpenClaw, when everyone started using it and the term spread widely.

This documentation explains what a harness actually is, how to build a harness around an AI agent, and — more importantly — why this harness concept exists in the first place.

## Understanding Harness Through a Car Analogy

Before diving into harness engineering for AI agents, it helps to understand a practical, everyday example: a car.

### The Engine Is the Core, But Not the Whole Car

A car is made up of many components — tires, a frame, a chassis, luxury interiors, a music system, a steering wheel. But if there's one thing that matters most in a car, it's the **engine**.

> **Key point:** The engine drives the entire car. If you remove the engine, it's really not a car anymore. The engine is the most important, most expensive, and most defining feature of that car — whether the car is comfortable, fast, slow, or fuel-efficient all depends on the engine.

### What Does an Engine Actually Do?

The engine takes some kind of input — for example, fuel plus air — and produces an output: **torque** (motion). That's the complete function of an engine: give it fuel and air, and it gives you torque.

### The Problem: An Engine Alone Has No Value

Here's the catch: can you actually sell just an engine in the market? If someone said, "I built an engine, and it costs ₹15 lakhs," would you buy it? As a normal consumer, the answer is no.

> **Key point:** A raw engine adds no value on its own. You can't sit in it, you can't drive it, and it can't move by itself. It's just a black box that needs fuel and air and produces torque. If that torque can't take you from point A to point B — if there's no seat, no controls — then even a ₹15 lakh engine is useless to you.

### How Car Companies Add Value Around the Engine

This is exactly where automobile engineers and companies like BMW and Ferrari come in. They build a whole layer of luxury and practicality around the engine:

1. **Platform/chassis**: A platform (chassis) is built, and the engine is placed on top of it as a part of that structure.
2. **Transferring torque to the wheels**: The torque produced by the engine is transferred to the wheels so they can rotate, using components like a **gearbox**, which gives the user control over how that torque is used.
3. **Cabin and seating**: A full body — the cabin — is built around the engine to provide a premium, comfortable experience, with features that have nothing to do with the engine itself (leather seats, three-row or two-row seating, etc.).
4. **Steering**: Connected to the front wheels so the direction can be changed — again, something the engine doesn't care about.
5. **Suspension**: Added for a smoother experience.
6. **Pedals**: Clutch, brake, and other controls are added on top.

> **Key insight:** The engine doesn't care about any of this — whether the seats are leather or basic, how many rows there are, or how the torque is being used. Its only job is to take fuel and air and produce torque. Everything else is built around it to make it controllable and practical.

Once all of this is added, a car that started as a ₹15 lakh engine can now be sold as a ₹25 lakh or ₹35 lakh car — because practicality has been added around it. This complete, sellable module is what a car actually is, with the engine being just one part of it.

## Applying This to AI Agents

Fast-forwarding to today's world of AI agents: the "engine" in this context is the **LLM (Large Language Model)** — models like GPT (from OpenAI) or Claude (from Anthropic).

> **Key point:** You can never sell a raw LLM directly in the market, because it is of no use on its own — just like a raw engine.

### What Does an LLM (the Engine) Actually Do?

If you think of the LLM as an engine:

- **Input**: Input tokens.
- **Output**: A probability distribution over possible next tokens.

The LLM returns a probability distribution, a softmax function is applied to it, and the next token is predicted. That's the entire "engine" of an LLM.

> **Key point:** Trying to sell this raw engine has no real use case, because LLMs are stateless — they have no memory, no way to orchestrate themselves, and no way to act on their own. They can't tweet, they can't do anything by themselves, because they have no "body." Just like an engine without a body can't move, an LLM without the surrounding structure can't function usefully.

## What Is a Harness?

To make an LLM useful, everything that needs to be built around it — the platform, the tires, the steering, the gearbox, the transfer of torque — has an equivalent in AI agent development. This set of surrounding components is known as the **harness**.

> **Definition:** In the car analogy, the platform is a harness, the tires are a harness, the gearbox is a harness, the cabin is a harness, the seats and steering are a harness, the music system and sunroof are harnesses. All of these add on top of the engine, but none of them can function without the engine. The harness is what adds unique value and practicality to the AI agent.

### Components of an AI Agent Harness

For AI agents, the harness includes:

- **Memory layer**: Around the LLM, giving it persistent context.
- **Observability**: Monitoring and tracking what the agent is doing.
- **Evals**: Evaluating agent performance.
- **Tools**: Built for the agent to interact with the world.
- **Agent loop**: Defining how the agent runs and orchestrates itself.
- **Orchestration logic**: Deciding what to do with output tokens, how many agents to run, and how they coordinate.
- **Guardrails**: Since agents can go in any direction and LLM outputs are never fully predictable, guardrails are needed to add predictability and safety.
- **Security layer**: Protecting the system around the LLM.
- **Context management**: Since LLMs have a limited context window, managing what goes in and out of that window is also part of the harness.

> **Key point:** As a developer, you never build the LLM itself — that requires enormous amounts of money and data, which is why large companies like Meta, OpenAI, and Anthropic build and train these models (the "engines"). As an application developer, your responsibility is to build the harness around it. No one today uses a pure LLM model directly, because it's of no practical use — just like a bare engine.

## ML Engineers vs. Application/Harness Engineers

Just as an engine has to be built by someone, LLMs are built by **ML engineers** — people interested in how layers work, how backpropagation works, and how fine-tuning works. But automobile engineers and car companies who build around the engine don't care about the internals of the engine itself — they want the best engine, but their job is to build the harness around it.

> **Key point:** Similarly, as a GenAI application developer, applied AI developer, or application developer, what you build around the LLM engine is the harness — and this is exactly what gives your product a unique perspective, a unique position, and a unique selling point.

## Real-World Example of Harness Components

An example from harness-building work includes components such as:

- **Data zen**: A data pipeline defining how data flows — where to pull data from and how to transform it through AI agents.
- **Role-based access control**: Managing permissions.
- **Memory layer**: How memories are stored and managed.
- **Secure zen**: How LLMs are secured.
- **Notifications**: Sending notifications from the LLM/agent to users.
- **Observability, evals, and tools**: Additional layers built around the core model.

> **Key point:** All of these are harnesses. The better the harness engineering, the better the resulting product will be.

## Claude as an Example: Engine vs. Harness

Looking at Claude's models — Opus, and Sonnet — these are the **engines** (the LLM models). Claude Code, the CLI tool used to interact with Claude, is a **harness** built on top of these engines.

> **Key point:** Anthropic built both the engine (the LLM models) and the harness (Claude Code). But as a user, you never directly use the raw engine — you use the harness layer built on top of it. You can also build your own harnesses around these engines.

## Defining Harness Engineering

> **Definition:** Harness engineering refers to the discipline of building the operational environment, tools, and constraints that make AI agents useful — not just coding agents, but any type of agent. It operates on the principle: **AI Agent = AI Model + Harness**.

Subsystems and components typically included in a harness are:

- Instructions (defining what the agent should do)
- State management (similar to memory)
- Verifications
- Scoped sessions

> **Key point:** Within the harness, you can build almost anything that adds value, premium quality, and practicality to the underlying model.
