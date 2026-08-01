# gRPC Overview: Benefits, Workflow, and Why REST Still Dominates

## Introduction

When it comes to building APIs, there are different ways to do it. One popular method is REST, but gRPC takes a different approach.

> **Definition:** gRPC is a high-performance, open-source framework developed by Google that allows developers to build fast and efficient APIs.

This documentation provides an overview of gRPC with simple examples, its primary benefits, and the reasons why it has not achieved the same level of popularity as REST APIs.

## What Is gRPC?

gRPC is a modern, high-performance framework that allows developers to build fast and efficient distributed systems.

One of the main advantages of gRPC over traditional REST APIs is efficiency. REST APIs typically use JSON or XML for data transmission, which can be slow and resource-intensive, whereas gRPC uses **Protocol Buffers**.

### What Are Protocol Buffers?

> **Definition:** Protocol Buffers (also known as Protobuf) is a language-agnostic format developed by Google. It is a way to write down information so that computers can understand it — a special language that computers use to talk to each other, making it easier for different programs to communicate and work together.

## Example: An Online Store with Microservices

To better understand gRPC and Protocol Buffers, consider a simplified example of an online store that allows customers to place orders, using two microservices: a **user service** and an **order service**, communicating with each other using gRPC and Protocol Buffers.

> **Note:** Communication via gRPC does not natively work out of the box with mobile apps and web browsers. This example focuses specifically on communication between microservices.

### Step 1: Define the Messages

The messages exchanged between the user service and order service are defined using Protocol Buffers. Two proto files are created to define the message structures:

- `user.proto` for the user service.
- `order.proto` for the order service.

> **Note:** This can be done in a single proto file, but for the sake of modularity, it is recommended to have separate proto files for each service.

### Step 2: Generate the gRPC Code

Next, gRPC code is generated based on the defined Protocol Buffers. The Protocol Buffers compiler (`protoc`), along with a gRPC plugin, is used to generate code for the programming language of choice — for example, Go code generated from `order.proto`.

## The Development Workflow

The overall workflow for building gRPC-based microservices follows these steps:

1. **Write the proto file**: Developers write the proto file, which defines the structure of the messages exchanged between microservices, specifying message fields, their types, and any required services or RPC methods.
2. **Generate code**: Developers use the Protocol Buffer compiler (`protoc`) along with language-specific plugins to generate code in their desired programming language (for example, Go, though many other languages are supported). This generated code includes classes, structs, or objects that represent the messages and services defined in the proto file.
3. **Implement server and client components**: Using the generated code, developers implement the server-side and client-side components of the microservices, writing the necessary logic to handle incoming requests, process data, and send responses.

### Automatic Serialization and Deserialization

The serialization and deserialization of data are automatically handled by the gRPC framework. When sending a message from one microservice to another, gRPC takes care of serializing the message into binary format and deserializing it on the receiving end.

> **Key point:** Developers do not need to write explicit code for serialization and deserialization.

### Communication Between Microservices

The gRPC framework also handles the communication and interaction between microservices:

- Developers utilize the generated code to establish communication channels between microservices.
- The server-side microservice exposes the defined RPC methods.
- The client-side microservice invokes those methods to communicate with the server.

> **Note:** In a complete microservices architecture, the client microservice may be implemented in a different programming language or project than the server microservice. In such cases, the protocol buffer code is generated separately for each programming language or project to enable communication between the client and server.

### Client Libraries for Non-Native Clients

If the client is a mobile app or any other application that doesn't have direct access to protocol buffer definitions, it typically uses a generated client library specific to the client's programming language.

> **Key point:** The client library provides an abstraction layer that encapsulates the gRPC communication details, allowing the client application to interact with the order service using more familiar, language-specific objects and methods. The client library takes care of serializing and deserializing the data between the client and the order service using the same Protocol Buffers format.

## Key Benefits of gRPC

### Compact, Efficient Data Serialization

gRPC uses the binary Protocol Buffer format for data serialization, which is much more compact and efficient than text-based formats like JSON used by REST.

> **Key point:** This results in smaller payload sizes and faster data transmission, leading to improved network efficiency and reduced bandwidth usage.

### Built on HTTP/2

gRPC leverages the HTTP/2 protocol out of the box, which supports:

- Multiplexing
- Request-response streaming
- Header compression

> **Key point:** These features enable concurrent requests, efficient streaming of data, and reduced latency, making gRPC a high-performance communication framework.

### Bidirectional Streaming

Due to its HTTP/2 foundation, gRPC also supports **bidirectional streaming**, allowing both the client and server to send multiple messages over a single RPC connection.

> **Key point:** This enables efficient real-time communication, such as chat applications or live data streaming, where both parties can continuously send and receive data.

### A Standardized, Contract-Driven Approach

While both REST APIs and gRPC are language-agnostic, gRPC provides a more standardized and integrated solution specifically tailored for building microservices. gRPC has its own set of tools and libraries designed specifically for working with gRPC services, offering features such as Protocol Buffers and code generation, which provide a structured and contract-driven approach to defining APIs.

## Why REST Still Dominates

Despite gRPC's many advantages, several reasons explain why engineers and companies continue to use REST extensively.

### REST Has Been Around Longer

REST APIs have been around for much longer than gRPC, so developers are generally more familiar with them. This makes it difficult to convince teams to switch to gRPC if they are already comfortable with REST.

### Learning Curve

Gaining proficiency in gRPC requires a certain level of knowledge and experience with concepts such as Protocol Buffers and remote procedure calls, which can represent a significant learning curve for some engineers.

### Limited Native Browser Support

gRPC is designed primarily for communication between microservices and does not natively support web browsers. REST APIs, on the other hand, can be easily consumed by web browsers, mobile apps, and other clients.

> **Key point:** This flexibility makes REST a popular choice. While there are workarounds for this limitation in gRPC, they can make it more difficult to use gRPC for front-end development.

### Less Mature Ecosystem

While tools and libraries are available for working with gRPC, the ecosystem is not as mature as that of REST APIs, which means it can be more difficult to find resources and support for working with gRPC.

## Closing Thoughts

While gRPC offers many benefits over REST APIs, several challenges may prevent it from being as widely used in certain contexts. Nevertheless, when exploring options for designing a robust back-end services communication architecture, gRPC deserves serious consideration — its unique capabilities can greatly enhance system performance and scalability.
