# Streaming Real-Time RAG Responses with Server-Sent Events (SSE)

## Introduction

When building a Retrieval-Augmented Generation (RAG) platform, a central engineering question is how to deliver AI responses in real time at scale.

The challenge is not only generating answers with a large language model (LLM). The platform must also stream those answers smoothly and reliably to thousands of concurrent clients without causing infrastructure costs to grow unnecessarily.

**Server-Sent Events (SSE)** provided a simple, lightweight, and scalable way to stream tokens from a RAG pipeline. Although WebSockets are often considered the default solution for streaming, SSE can be a better fit when the application only needs server-to-client communication.

## Why Streaming Is Needed

Consider a RAG-powered assistant:

1. A user submits a question.
2. The system retrieves relevant context from a vector database.
3. The LLM generates an answer using that context.

If the system waits until the entire response has been generated before returning anything, the user may face a blank screen for more than 10 seconds. This creates a poor user experience.

### Token Streaming

With token streaming, the system sends response content as the LLM generates it. Users see words appear progressively, similar to streamed ChatGPT responses.

This makes the response feel much faster and more interactive, even when the total time required to generate the complete answer does not change.

> **Key point:** Token streaming dramatically improves perceived speed, even if total latency remains unchanged.

## Choosing a Streaming Protocol

Both WebSockets and Server-Sent Events were considered for token streaming.

| Aspect | WebSockets | Server-Sent Events (SSE) |
|---|---|---|
| Communication direction | Bidirectional | Unidirectional: server to client |
| Setup and management | Requires a more complex setup and persistent connection management | Requires minimal setup |
| Infrastructure | Often involves heavier infrastructure | Built on HTTP and comparatively lightweight |
| Browser support | Requires socket-oriented handling | Natively supported in modern browsers through `EventSource` |
| Suitable use case | Versatile, bidirectional communication | Push-style updates such as token streaming |

Because the RAG use case only required token delivery from the server to the client, SSE was the appropriate choice. It was simpler to implement, easier to debug, and worked well with the existing infrastructure.

> **Decision principle:** If the application does not need bidirectional communication, SSE can avoid much of the complexity associated with WebSockets.

## How SSE Works

SSE uses a simple text-based protocol over HTTP. The server keeps an HTTP connection open and sends messages as newline-delimited strings.

### Server Response Format

A server can send generated tokens as SSE `data:` messages:

```text
data: {"token": "Hello"}

data: {"token": " world"}

data: {"token": "!"}
```

### Client-Side Listener

A browser client can listen to the stream with the native JavaScript `EventSource` API:

```javascript
const eventSource = new EventSource("/api/stream");

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("Received token:", data.token);
};
```

At a high level, this is all that is required: no socket handshake, no custom client libraries, and no heavy client-side setup.

## RAG Architecture with SSE

The following flow describes how LLM responses can be streamed through SSE.

### 1. Client Request

The user submits a query to a RAG API endpoint, such as `/chat`.

### 2. Retriever Step

The API retrieves relevant documents from a vector database. Depending on the environment, this may use Pinecone, Weaviate, or another vector database.

### 3. LLM Generation with Token Callbacks

The platform streams tokens directly from the LLM provider SDK. Examples include OpenAI with a `stream=True` flag or a local model that exposes token callbacks.

### 4. SSE Stream to Client

Each generated token is packaged into an SSE `data:` message and flushed to the client immediately.

### 5. Completion Signal

Once generation finishes, the server sends a `[DONE]` event so that the client can close the stream gracefully.

## Scaling Considerations

Production use of SSE introduces several operational considerations.

### Connection Limits

SSE connections are long-lived HTTP requests. Load balancers, such as Nginx, Cloudflare, or ALB, must be configured to support thousands of concurrent keep-alive connections.

### Backpressure

Tokens should be flushed as they arrive. However, buffering must be handled carefully when the LLM generates tokens faster than the network can transmit them to the client.

### Resilience

If the connection drops, the browser's `EventSource` automatically retries. This provides a significant stability benefit compared with manually managing WebSocket reconnections.

### Stateless Scaling

Because SSE runs over HTTP, multiple stateless API servers can be scaled horizontally. This avoids the need for sticky sessions or socket brokers.

## Lessons Learned

- **SSE is underrated.** Developers often choose WebSockets immediately, but SSE is easier to implement, debug, and scale when only server-to-client streaming is required.
- **User experience matters.** Token streaming is not merely a nice-to-have feature; it changes how fast and interactive a RAG system feels to users.
- **Infrastructure awareness is essential.** Long-lived connections require deliberate configuration of load balancers, timeouts, and monitoring. They should be treated differently from normal HTTP requests.

## Final Thoughts

Building a scalable RAG system involves more than retrieval and generation. Delivering the generated response effectively is equally important.

SSE offers a combination of simplicity and scale: it is a lightweight mechanism for streaming AI responses in real time without overengineering the stack.

When deciding between WebSockets and SSE for an LLM-powered application, first determine whether bidirectional communication is actually required. If it is not, SSE may remove substantial implementation and infrastructure complexity.
