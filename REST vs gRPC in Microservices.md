# Interprocess Communication in Microservices: Styles, APIs, and Messaging Formats

## Introduction

When building applications with microservices, one of the biggest decisions to face is how services will communicate — known as **interprocess communication (IPC)**. Choosing the right IPC strategy can make an application smoother, faster, and easier to maintain.

There are two main ways microservices can talk to each other:

- **Synchronous communication**: HTTP-based approaches like REST or gRPC, where services expect immediate responses.
- **Asynchronous communication**: Messaging protocols like AMQP or Kafka, where responses aren't immediate, offering more flexibility.

The right messaging format also matters — human-readable formats like JSON or XML, or efficient binary formats like Avro or Protocol Buffers.

This documentation focuses first on crucial design choices: interaction styles, the importance of clearly defining APIs (especially with an API-first approach), how clients invoke services using messages, broker-based versus brokerless messaging architectures, and core messaging concepts.

## Interaction Styles

IPC is basically how microservices talk to each other. Before jumping into specific technologies, it helps to think clearly about interaction styles first.

> **Key point:** Knowing how services and their clients interact makes choosing the right IPC solution much easier — it helps focus on what's really important rather than getting lost in technical details. This choice also has a big impact on an app's availability.

Interactions can be categorized along two dimensions:

1. **How many services are involved**: One-to-one interactions (one client talks directly to one service) versus one-to-many interactions (one client's request is handled by multiple services).
2. **Timing**: Synchronous interactions, where the client sends a request and expects a quick reply (like a phone call waiting for someone to pick up), versus asynchronous interactions, where the client sends a request but doesn't wait around for an immediate reply (like sending an email and continuing with your day).

### One-to-One Interactions

- **Request/response**: The client sends a request and expects a quick answer, possibly pausing while waiting. This approach tends to tightly couple services.
- **Asynchronous request/response**: The client sends a request but doesn't pause its work — it keeps going, knowing the service might take some time to respond.
- **One-way notifications**: The client sends a message to a service but doesn't expect any reply at all; it's purely informational.

> **Important note:** Even when using messaging, an interaction can still be synchronous — for example, a service might send a message and still pause waiting for a reply. Using messaging doesn't always mean services are loosely coupled.

### One-to-Many Interactions

- **Publish/subscribe (pub/sub)**: The client sends out a notification, and any number of services can listen in and react.
- **Publish/async responses**: The client broadcasts a request and then waits a certain amount of time for multiple responses from interested services.

> **Note:** Most real-world applications use a combination of these interaction styles.

## API-First Design

No matter which IPC style is chosen, clearly defining APIs using an **Interface Definition Language (IDL)** is crucial. This is where **API-first design** comes in.

> **Definition:** In API-first design, a clear definition of the APIs is created first, then shared with client developers — the people who will actually use those APIs. Feedback is gathered and adjustments are made, and only once everyone is happy does the team start building the services.

### Why API-First Matters

This ensures that what is built actually matches what clients need. Even in smaller projects, skipping the API-first design approach can lead to confusion and frustration.

> **Real-world scenario:** Backend developers working in Java and front-end engineers using AngularJS may both confidently say they are done, but when they try running the app, nothing works — because the REST APIs weren't clearly defined beforehand. Even though each side did their part, the pieces couldn't communicate properly.

That's exactly why an API-first approach isn't just nice to have — it's essential. Defining APIs clearly from the start saves everyone a lot of headaches and keeps the whole team aligned from day one.

### API Definitions Vary by IPC Mechanism

The API definition varies depending on the IPC mechanism used:

- With **messaging**, the API mainly involves specifying message channels, message types, and formats.
- With an **HTTP-based API**, the definition includes URLs, HTTP verbs like GET or POST, and the formats for requests and responses.

## Choosing a Message Format

At the heart of IPC are messages, which typically carry data around. Choosing a message format is a critical decision because it impacts an API's performance, ease of use, and flexibility to evolve over time.

> **Key point:** No matter which IPC method is used, it's essential to pick a message format that works well across different programming languages. Services might be written in Java today, but could use Go, Python, or JavaScript tomorrow — so a language-neutral format is important. Avoid relying on language-specific serialization, such as Java serialization.

There are two broad types of message formats: text-based and binary.

### Text-Based Formats (JSON, XML)

Text-based formats like JSON and XML are popular because they are easy for humans to read and are self-describing. For example, a JSON message has clearly named properties, making it easy for services to pick only what they need and ignore the rest.

> **Key point:** This flexibility means changes to messages often don't break existing services.

To clearly define the structure of these messages, developers use schemas — XML has XML Schema, and JSON has a similar standard called JSON Schema.

> **Why schemas matter:** Schemas don't just document message structure — they also enable automatic validation of incoming messages, ensuring they are exactly what services expect.

### Binary Formats (Protocol Buffers, Avro)

Binary formats like Protocol Buffers and Avro aren't human-readable like JSON or XML, but they are incredibly efficient. The message structure is defined clearly up front using an Interface Definition Language (IDL), and a compiler generates the code for serialization and deserialization.

> **Key advantage:** Binary formats essentially force an API-first design, meaning APIs are clearly defined from the start. With strongly typed languages, the compiler will also catch mistakes early, ensuring clients use the API correctly.

**Protocol Buffers vs. Avro:**

| Format | Field Handling |
|---|---|
| Protocol Buffers | Tags each field, letting consumers skip unknown fields easily |
| Avro | Requires the consumer to know the schema ahead of time |

> **Key point:** This makes Protocol Buffers somewhat easier to manage as an API evolves over time. Choosing the right message format isn't just a technical detail — it directly affects how easy it will be to grow and maintain APIs down the line.

## Remote Procedure Invocation (RPI)

With message formats covered, the next step is looking at a specific IPC mechanism that transports messages: the **Remote Procedure Invocation (RPI)** pattern.

> **Definition:** RPI is a way for services to talk synchronously — think of it as a phone call between services. One service makes a request, and the other answers immediately in real time.

### How RPI Works

1. The client service sends a request through a **proxy interface** — an adapter that handles the communication details.
2. This proxy forwards the request to a target service, which processes it.
3. The target service sends back a response through the same proxy, returning the results to the client's logic.

Two of the most popular implementations of this pattern are **REST** and **gRPC**.

## REST APIs

REST has become hugely popular because it's simple, widely used, and leverages the familiar HTTP protocol. REST APIs treat everything they interact with — customers, orders, products — as resources, manipulated using standard HTTP verbs (GET for reading, POST for creating, PUT for updating, and so on).

**Example:**

- `POST /orders` → create a new order
- `GET /orders/{orderId}` → retrieve details about a specific order

### Benefits of REST APIs

- Simple and familiar — most developers already understand them.
- Easy to test directly from browsers, tools like Postman, or command-line utilities such as curl.
- Naturally fits the request-response communication style.
- Works well through firewalls and doesn't require extra setup.
- No need for an intermediate broker, keeping the architecture simpler.

### Downsides of REST APIs

- Only supports synchronous request-response interactions.
- Without an intermediary broker to handle messages, both the client and services need to stay available during the entire interaction, potentially impacting availability.
- Clients have to know exactly where service instances are located, requiring service discovery.
- Retrieving multiple resources efficiently in one request can be challenging.
- Mapping complex business operations into simple HTTP verbs isn't always straightforward.

> **Note:** Despite these drawbacks, REST remains the most common standard today. Alternatives worth considering include GraphQL (for efficient data fetching) and gRPC (for robust synchronous communication).

## gRPC

One challenge with REST is that HTTP provides only a limited set of verbs (GET, POST, PUT, etc.), making it sometimes tricky to clearly map complex business operations. This is exactly where gRPC shines.

> **Definition:** gRPC is a robust, binary message-based framework designed by Google specifically for building cross-language services and clients. Because it uses a binary message format called Protocol Buffers in a `.proto` file, it pushes developers into adopting an API-first design.

### How gRPC Works

With gRPC, APIs are clearly defined up front using Protocol Buffers — Google's structured, language-neutral mechanism. Once defined, the Protocol Buffer compiler generates the necessary code — client-side stubs and server-side skeletons — for multiple languages like Java, C++, Node.js, and Go.

**Example RPC methods:**

- **CreateOrder**: A unary call, where the client sends a request and gets a single response. Ideal for operations like placing an order.
- **TrackOrder**: Uses server streaming, meaning the client sends one request and receives a continuous stream of updates — perfect for live order status updates like "preparing," "shipped," or "delivered."

> **Key point:** This demonstrates gRPC's strength — it supports both standard request-response and advanced patterns like streaming, all with strong typing and efficient data transfer.

### Benefits of gRPC

- High performance, thanks to HTTP/2 and Protocol Buffers — much faster and more compact than REST with JSON.
- Supports streaming, enabling real-time communication via client, server, or bidirectional streaming.
- Generates reliable client and server code in almost any language, preventing integration bugs and making API evolution safer and smoother.

### Downsides of gRPC

- Doesn't work out of the box in browsers — often requires gRPC-Web or an API gateway to bridge the gap.
- Not all firewalls or legacy infrastructure support HTTP/2, which could be a blocker in some environments.

## Looking Ahead

gRPC offers powerful features like streaming, strong typing, and high performance, while REST remains a reliable and familiar option. However, synchronous communication isn't always the right fit — especially when services need to work independently without waiting on each other.

> **Coming next:** Asynchronous messaging — how services can communicate by exchanging messages through brokers like RabbitMQ or Kafka, and even through brokerless setups, along with messaging patterns like publish/subscribe and message queues that help build resilient, decoupled systems.
