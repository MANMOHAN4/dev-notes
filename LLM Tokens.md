# Understanding LLM Tokens: Compute, Cost, and Optimization

## Introduction

When you call an LLM, it looks like you are sending text, but what you are actually sending is compute. Every token you send, and every token you get back, is work that someone's GPU has to do. That work shows up in your bill, in your latency, in everything.

Most engineers think about LLMs in terms of words, characters, or messages. But the model doesn't see any of that , the model sees **tokens**.

> **Key point:** Tokens are what you actually pay for. Tokens decide how fast your application runs , not words, not messages. Tokens.

This documentation covers:

- What tokens actually are.
- Why they cost what they cost.
- The journey a token takes when you call an LLM.
- How to spend tokens efficiently so your bills stay predictable as you scale.

## What Is a Token?

The best way to understand tokens is to use a tokenizer , a tool that takes any piece of text and shows how the model splits it into tokens.

### Example 1: "The quick brown fox jumps over the lazy dog"

This sentence has nine words. If tokens were words, you would expect nine tokens , and in this case, it is close to nine, almost one token per word.

### Example 2: "tokenizationally"

This single word breaks into **five tokens**, not one. The model has never seen "tokenizationally" often enough to give it a dedicated token, so it breaks the word into smaller pieces it does know. Each piece costs separately.

### Example 3: A UUID

A single UUID can become **20 tokens** from what looks like one "word." The model looks at random characters and dashes and has no efficient way to compress them.

> **Key point:** Every UUID in your prompt is 20 tokens of tax. If your retrieval layer pulls in five documents, each with a few UUIDs at the top, that is 100 tokens of pure metadata before you have said anything useful.

### Example 4: A Line of Python Code

A single line of Python code, roughly 40 characters, can become **15 tokens**. Code is dense in tokens because every underscore, colon, bracket, and asterisk gets counted.

> **Key point:** This is why pasting a stack trace into a prompt costs much more than pasting the same length of English text. The model is not reading the meaning , it is counting symbols.

### Core Idea

- **Words** are a human unit.
- **Characters** are a transport unit.
- **Tokens** are the model's compute unit , and compute is what you pay for.

What looks short to you can be long to the model. What looks long can be surprisingly short. If you're building anything with LLMs, you have to learn to look at your prompts the way the model looks at them: in tokens, not words.

## Why Tokenization Works This Way

Tokenizers are **learned**. Someone takes a huge pile of text and runs an algorithm that figures out which patterns show up most often.

- Common words like "the," "is," and "and" get their own token because they show up everywhere. Giving them a short representation saves compute later.
- Rare words don't get that treatment. There is no token for "tokenizationally" because the algorithm never saw it enough times to justify one. When you use that word, the tokenizer falls back to breaking it into smaller, more common chunks.

> **Technical term:** This process is called **Byte Pair Encoding (BPE)**. The rule to remember: common patterns are cheap, rare patterns are expensive. The tokenizer decides what counts as common based on what it was trained on , mostly English text from the internet.

### Three Practical Consequences

1. **Different models have different tokenizers.** Claude's tokenizer is not the same as GPT-4's. Llama's is different. The same exact sentence will count differently on each one.
2. **Non-English text usually tokenizes worse.** Languages like Hindi, Mandarin, and Arabic often take two to three times more tokens per character than English. If you are building for a global audience, your bill in some languages is fundamentally higher.
3. **Code, logs, JSON, and UUIDs explode the token count.** Anything with lots of punctuation and random-looking strings costs significantly more. Pasting a stack trace into a prompt costs way more than pasting the same amount of English prose.

> **Operational lesson:** What looks short to you can be long to the model, and what looks long can be short. You can't eyeball it , you have to measure.

## What Happens During an API Call

When your code calls a model like Claude or GPT-4, a specific sequence of events occurs. This section introduces four key terms: **inference**, **prefill**, **decode**, and **context window**.

### Training vs. Inference

There are two phases in a model's life:

- **Training**: When the model learns. Billions of pages of text are run through huge clusters of GPUs over months, producing a model that has learned patterns from that data. Training happens once, costs tens of millions of dollars, and takes months of GPU time.
- **Inference**: When the model runs. Every time you call the API, every time someone types a message to a chatbot, every time your application sends a prompt , that's an inference. The model is executing what it learned, not learning anything new.

> **Key point:** Training is a one-time capital expense. Inference is what you pay for every single day, on every single request. When talking about cost, latency, and scaling, you are always talking about inference.

### The Five Steps of Inference

When you call an LLM completion endpoint, five things happen:

1. **Tokenization**: Your string of text hits the provider's server, and the first thing that happens is it runs through the tokenizer. The text becomes a sequence of token IDs (numbers). For example, "Hello world" might become numbers like 15496 and 11 and 1917 , that's what the model actually consumes, not text.
2. **Embedding**: Each token ID gets mapped to a high-dimensional vector , a list of a few thousand numbers. This is how the model represents meaning internally, encoding what a word means based on its surrounding context.
3. **Prefill**: The model reads your entire input prompt all at once. All the tokens go through the model in parallel, building up the internal state needed to start generating a response. Prefill is fast because it's parallel , the model performs thousands of computations simultaneously on different tokens. The time you wait between hitting enter and seeing the first character of the response is mostly prefill time.
4. **Decode**: The model generates the response. The model can only generate one token at a time , each new token depends on all the previous ones, so decode is sequential and cannot be parallelized. Every single output token requires a full pass through the model (hundreds of billions of parameters firing) to produce one more token, and this repeats until the response is complete.
5. **Detokenization**: The output token IDs get mapped back to text and streamed to the client. This step is fast and basically free.

### Why Output Tokens Cost More

Input tokens are processed in parallel in one shot during prefill. Output tokens are processed one at a time, sequentially, during decode.

> **Key point:** This is why output tokens cost three to five times more than input tokens on most providers. It's not a pricing decision , it's a physics decision. Generating tokens is fundamentally slower and more expensive than reading them.

This also explains why two API calls that look identical to a user can take wildly different amounts of time. A short question with a short answer is a cheap workload. A call with a 5,000-token document stuffed into the prompt, asking for a long answer, is a completely different (and more expensive) workload to the GPU , even though it looks the same to the user.

## Common LLM Problems That Are Actually Token Problems

Three common issues that engineers often blame on the model are actually token-management problems.

### The Model Forgets a Constraint

A system prompt works perfectly in testing but is sometimes ignored in production. What likely happened: the context got crowded. The system prompt is at the top, then long conversation history is added, then five retrieved documents are stuffed in. By the time the model generates its response, the original instruction is buried thousands of tokens earlier.

> **Key point:** Attention dilutes. The instruction is still technically there but is no longer salient.

The same pattern explains:

- **Hallucination**: Not enough tokens spent on grounding.
- **Inconsistency**: Context bloat dilutes the relevant signal.

> **Takeaway:** When models forget, hallucinate, or behave inconsistently, don't blame the model first. Audit your context. Ask what is actually in there, and whether each token is earning its place. Spend your tokens on purpose.

## Understanding LLM Cost

Every LLM has two prices: the price to read your input and the price to generate the output. Outputs cost three to five times more per token than inputs, because of the decode process described earlier , every output token is a full sequential forward pass through the model, while inputs are processed in parallel during prefill.

> **Key point:** Verbosity is not a style choice , it's a burn rate. A model that answers in two paragraphs when one sentence would do costs three to five times more per response.

When building applications, decide the answer shape yourself: set max tokens, use structured outputs, and don't let the model write whatever length it wants.

## Four Levers to Reduce Token Cost

There are four levers to control LLM cost, and most teams pull none of them.

### 1. Trim What You Send

Your system prompt goes out on every single request, so every word left in it is a tax paid repeatedly.

> **Key point:** A prompt that is twice as long is not a one-time cost , it is double the input bill on every call, forever.

Go through your prompts and cut anything the model doesn't actually need. On the output side, cap max tokens to the smallest number that still works. Every token you don't generate is money saved.

### 2. Prefix Caching

Prefix caching is one of the biggest hidden cost levers in the entire LLM stack. Most engineers don't even know it exists, but teams that use it often cut costs by 80 to 90%.

How it works:

- Before the model generates a single token, it has to read your entire prompt and do the prefill work.
- If you send the same system prompt on every request, you're paying for that same prefill work again and again.
- Prefix caching stops this waste: the first time a provider processes your prompt, it stores the prefill state.
- On your next request, if the start of the prompt matches, the provider skips recomputation and loads the cached state, only processing the new part (your actual user input).

> **Key point:** Cached tokens are not free, but they are close , billed at a small fraction of the normal input price. Send the same long prompt twice, and from the second call onward it is almost all savings.

Both Anthropic and OpenAI support prefix caching, but most teams still aren't using it properly. If your app keeps sending a long system prompt, this is one of the easiest wins available.

### 3. Model Tiering

Not every request needs your most expensive model. Simple tasks such as classification, field extraction, request routing, or short summaries run perfectly well on a smaller, cheaper model. You only need the frontier model for hard reasoning tasks.

> **Analogy:** This is the same idea as a CPU cache hierarchy , you don't go to main memory for every operation because a faster, cheaper layer usually already has what you need. Match the model to the difficulty of the task.

### 4. Provider Selection

The same model can cost wildly different amounts depending on who is serving it. For example, Llama 3.3 is an open-weights model hosted by multiple providers, including Together AI, Fireworks, Groq, and DeepInfra. Same weights, same model, same outputs , but the price per token and latency can differ significantly between providers.

> **Key point:** Picking a model is only half the decision. You also have to pick the provider running it. Most teams wire up one SDK on day one, never revisit it, and quietly overpay for months.

## Solving Provider Fragmentation: OpenRouter

Shopping across providers for every model, integrating multiple APIs, normalizing responses, and handling different error formats is impractical for most teams. This is the problem a unified inference layer solves.

**OpenRouter** is one API for over 300 models across every major provider. It speaks the OpenAI chat completion schema, so no new SDK is required , you use the same client library you already have and simply point it at OpenRouter's base URL.

### Getting Set Up

1. Go to openrouter.ai and select "Get API Key."
2. Set an expiration date.
3. Copy the key and add it to your environment file.

### Example Workflow

A simple ~50-line Python script that takes a chunk of code and asks an LLM to summarize it in one sentence can be run with OpenRouter using the standard OpenAI client library. The only change needed is the base URL , pointing it at OpenRouter instead of OpenAI.

From there, only the model string changes between runs:

- **Run 1 , Claude Sonnet**: Used as the baseline, tracking input token count and cost.
- **Run 2 , GPT-4o**: Same input text, but a different input token count and bill, because different models use different tokenizers.
- **Run 3 , DeepSeek V3.2**: Frontier-quality output at a fraction of the cost of Claude.
- **Run 4 , Llama 3.3 70B with a ":floor" suffix**: This suffix instructs OpenRouter to route to the cheapest provider currently running the model.
- **Run 5 , Same model with a ":nitro" suffix**: Instead of the cheapest provider, OpenRouter routes to the fastest one available, resulting in a real latency improvement.

> **Key point:** OpenRouter passes provider pricing straight through , there is no per-token markup added on top.

### Additional OpenRouter Features

- **Fallback chains**: You can configure a primary model, a backup model, and a second backup. If your primary provider has an outage, traffic shifts mid-flight without users noticing.
- **Privacy**: OpenRouter offers zero-data-retention providers if needed, and does not log prompts or responses by default.

Free credits are available to get started, and documentation is available at openrouter.ai/docs.

## Closing Thoughts

At the core, all of these decisions come down to which tokens you choose to include and which tokens you refuse to carry.

Engineers who build with this in mind create systems that scale predictably , their bills stay predictable and latency stays tight as the application grows. Engineers who don't often learn this the hard way in an incident review, discovering that their compute budget was spent on punctuation, IDs, and blobs rather than meaningful content.
