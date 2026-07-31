# Vector Databases: Concepts, Architecture, and Use Cases

## Introduction

A **Vector Database** is a specialized database system designed for efficiently indexing, querying, and retrieving high-dimensional vector data. These systems enable advanced data analysis and similarity-search operations that extend well beyond the traditional, structured query approach of conventional databases.

## Why Use a Vector Database?

### The Data Flood Is Real

In 2024, the world is drowning in unstructured data , images, text, and audio , that doesn't fit into neatly organized tables. Yet there is still a need to tap into the value within this chaos, with almost 330 million terabytes of data being created each day.

> **Key point:** Traditional databases, even with extensions that provide some vector handling capabilities, struggle with the complexities and demands of high-dimensional vector data.

Handling vector data is extremely resource-intensive. A traditional vector is around 6KB, and scaling to millions of vectors can demand substantial system memory and computational resources , which is at least very challenging for traditional [OLTP](https://www.ibm.com/topics/oltp) and [OLAP](https://www.ibm.com/topics/olap) databases to manage.

### What Vector Databases Enable

Vector databases allow understanding of the context or conceptual similarity of unstructured data by representing it as vectors, enabling advanced analysis and retrieval based on data similarity.

- **Recommendation systems**: Vector databases can analyze user behavior and item characteristics to suggest products or content with a high degree of personal relevance.
- **Search engines and research databases**: They enhance the user experience by providing results that are semantically similar to the query, rather than relying solely on the exact words typed into the search bar.

## What Is Vector Data?

To understand vector databases, it helps to first define a "vector" or "vector data."

> **Definition:** Vectors are a numerical representation of some type of complex information.

- For **textual data**, a vector encapsulates the nuances of language, such as semantics and context.
- For an **image**, vector data encapsulates aspects like color, texture, and shape. The dimensions relate to the complexity and the amount of information each image contains.

Each pixel in an image can be seen as one dimension, as it holds data (like color intensity values for red, green, and blue channels in a color image). Even a small image with thousands of pixels translates to thousands of dimensions.

> **Note:** From this point on, "high-dimensional data" refers to data that contains a large number of data points (pixels, features, semantics, syntax).

The creation of vector data , so it can be stored in a vector database , is primarily done through **embeddings**.

## How Do Embeddings Work?

Embeddings translate high-dimensional data into a more manageable, lower-dimensional vector form that's more suitable for machine learning and data processing applications, typically through neural network models.

In creating dimensions for text, the process involves analyzing the text to capture its linguistic elements. Transformer-based neural networks like **BERT** (Bidirectional Encoder Representations from Transformers) and **GPT** (Generative Pre-trained Transformer) are widely used for creating text embeddings.

Each layer of these networks extracts different levels of features, such as context, semantics, and syntax. The final layers of the network condense this information into a vector , a compact, lower-dimensional representation of the original data that still retains the essential information.

## Core Functionalities of Vector Databases

### Indexing

Finding a specific face in a massive crowd photo illustrates the challenge vector databases face when dealing with tons of high-dimensional vectors.

Imagine dividing the crowd into smaller groups based on hair color, then eye color, then clothing style , each layer gets you closer to who you're looking for. Vector databases use similar multi-layered structures called **indexes** to organize vectors based on their "likeness."

> **Key point:** This way, finding similar vectors becomes a quick hop across related groups, instead of scanning every item one by one.

Different indexing methods exist, each with its own strengths:

- **[HNSW](https://qdrant.tech/articles/filtrable-hnsw/)** (Hierarchical Navigable Small World) balances speed and accuracy, like a well-connected network of shortcuts in the crowd.
- Others, like **IVF** (Inverted File Index) or **Product Quantization**, focus on specific tasks or memory efficiency.

### Binary Quantization

**Quantization** is a technique used for reducing the total size of the database. It works by compressing vectors into a more compact representation at the cost of some accuracy.

> **Definition:** [Binary Quantization](https://qdrant.tech/articles/binary-quantization/) is a fast indexing and data compression method used by Qdrant. It supports vector comparisons, which can dramatically speed up query processing times , up to 40x faster.

Think of each data point as a ruler. Binary quantization splits this ruler in half at a certain point, marking everything above as "1" and everything below as "0." This [binarization](https://deepai.org/machine-learning-glossary-and-terms/binarization) process results in a string of bits representing the original vector.

> **Key point:** This "quantized" code is much smaller and easier to compare. Especially for OpenAI embeddings, this type of quantization has proven to achieve a massive performance improvement at a lower cost of accuracy.

### Similarity Search

> **Definition:** [Similarity search](https://qdrant.tech/documentation/concepts/search/) allows searching not by keywords but by meaning. This enables searches such as finding similar songs that evoke the same mood, finding images that match an artistic vision, or exploring emotional patterns in text.

**How it works:**

1. When a user queries the database, the query is also converted into a vector (the query vector).
2. The [vector search](https://qdrant.tech/documentation/overview/vector-search/) starts at the top layer of the HNSW index, where the algorithm quickly identifies the area of the graph likely to contain vectors closest to the query vector.
3. The algorithm compares the query vector to all others, using metrics like "distance" or "similarity" to gauge how close they are.
4. The search then moves down progressively, narrowing down to more closely related vectors, with the goal of narrowing down the dataset to the most relevant items.
5. Once the closest vectors are identified at the bottom layer, these points translate back to actual data (like images or music), representing the search results.

### Scalability

Vector databases often deal with datasets comprising billions of high-dimensional vectors. This data isn't just large in volume but also complex in nature, requiring more computing power and memory to process.

> **Key point:** Scalable systems can handle this increased complexity without performance degradation, achieved through a combination of distributed architecture, dynamic resource allocation, data partitioning, load balancing, and optimization techniques.

Systems like **Qdrant** exemplify scalability in vector databases, leveraging Rust's efficiency in memory management and performance to handle large-scale data with optimized resource usage.

### Efficient Query Processing

The key to efficient query processing is linked to indexing methods, which enable quick navigation through complex data structures. By mapping and accessing the high-dimensional vector space, HNSW and similar indexing techniques significantly reduce the time needed to locate and retrieve relevant data.

Other techniques used to improve performance include:

- **Handling computational load and parallel processing**, especially when managing multiple simultaneous queries.
- **Strategic caching**, which stores frequently accessed data to facilitate quicker retrieval for subsequent queries.

### Filtering with Metadata

Filters use metadata to refine search queries within the database. For example, in a database containing text documents, a user might want to search for documents not only based on textual similarity but also filter results by publication date or author.

> **Key point:** When a query is made, the system can use both the vector data and the metadata to process the query. The database doesn't just look for the closest vectors , it also considers additional criteria set by metadata filters, creating a more customizable search experience.

### Data Security and Access Control

Vector databases often store sensitive information, which could include personal data in customer databases, confidential images, or proprietary text documents. Ensuring data security means protecting this information from unauthorized access, breaches, and other cyber threats.

At Qdrant, security mechanisms include:

- User authentication
- Role-based access control
- Attribute-based access control
- Encryption for data at rest and in transit
- Keeping audit trails
- Advanced database monitoring and anomaly detection

## Architecture of a Vector Database

A vector database is made up of multiple different entities and relations. The following is a high-level overview of Qdrant's terminology and how it fits into the larger picture.

| Component        | Description                                                                                                                                                                                                                                                                                                         |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Collections      | A [named set of data points](https://qdrant.tech/documentation/concepts/collections/), where each point is a vector with an associated payload. All vectors within a collection must have the same dimensionality and be comparable using a single metric.                                                          |
| Distance Metrics | Metrics used to measure the similarity between vectors. The choice of distance metric is made when creating a collection, depending on the nature of the vectors and how they were generated, considering the neural network used for encoding.                                                                     |
| Points           | Each [point](https://qdrant.tech/documentation/concepts/points/) consists of a vector and can also include an optional identifier (ID) and payload. The vector represents the high-dimensional data, and the payload carries metadata information in JSON format, giving the data point more context or attributes. |
| Storage Options  | Two primary storage options exist: in-memory storage keeps all vectors in RAM, allowing the highest speed in data access since disk access is only required for persistence; Memmap storage creates a virtual address space linked with the file on disk, balancing memory usage and access speed.                  |
| Clients          | Qdrant supports various programming languages for client interaction, such as Python, Go, Rust, and TypeScript, allowing developers to connect to and interact with Qdrant using their preferred language.                                                                                                          |

## Vector Database Use Cases

If the use cases for vector databases had to be summarized in a single word, it would be "**match**." They are great at finding non-obvious ways to correspond or "match" data with a given query , whether through similarity in images, text, user preferences, or patterns in data.

### Examples of Use Cases

- **Personalized recommendation systems**: Analyze and interpret complex user data, such as preferences, behaviors, and interactions. For example, on Spotify, if a user frequently listens to the same song or skips it, the recommendation engine takes note of this to personalize future suggestions.
- **Semantic search**: Allows systems to capture the deeper semantic meaning of words and text. In modern search engines, if someone searches for "tips for planting in spring," the system tries to understand the intent and contextual meaning behind the query rather than just matching the words themselves. An example is Qdrant's [vector search engine for startups](https://demo.qdrant.tech/).
- **Fraud detection and anomaly analysis**: Used in sectors like finance and cybersecurity to detect anomalies and potential fraud.
- **Content-Based Image Retrieval (CBIR)**: Retrieves images by comparing vector representations rather than metadata or tags.

> **Key point:** These are just a few examples. The ability of vector databases to "match" data with queries makes them essential for multiple types of applications. Additional [use case examples](https://qdrant.tech/use-cases/) are available for further exploration.
