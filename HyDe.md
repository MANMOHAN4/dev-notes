# How to Use HyDE for Better LLM RAG Retrieval

## Introduction

Large Language Models (LLMs) can be improved by giving them access to external knowledge through documents. The basic Retrieval-Augmented Generation (RAG) pipeline consists of a user query, an embedding model that converts text into embeddings (high-dimensional numerical vectors), a retrieval step that searches for documents similar to the user query in the embedding space, and a generator LLM that uses the retrieved documents to generate an answer.

> **Key point:** In practice, the RAG retrieval part is crucial. If the retriever does not find the correct document in the document corpus, the LLM has no chance to generate a solid answer.

### The Core Problem

A problem in the retrieval step can arise because the user query is often a very short question , with imperfect grammar, spelling, and punctuation , while the corresponding document is a long passage of well-written text that contains the desired information. Query and document typically differ significantly in length and format, which makes direct similarity matching between them difficult.

**HyDE** is a proposed technique to improve the RAG retrieval step by converting the user question into a hypothetical document. This documentation explains the HyDE technique and how and when to use it to improve a RAG pipeline.

## HyDE Retrieval

**Hypothetical Document Embeddings (HyDE)** were first proposed in the paper "Precise Zero-Shot Dense Retrieval without Relevance Labels" in 2022.

> **Definition:** The goal of HyDE is to transform the user query into a "document" so that the retriever has an easier task.

### How HyDE Works

A pre-trained LLM transforms the user query into a hypothetical fake document. The retriever then uses the fake document to search for similar real documents in the knowledge database.

HyDE uses an off-the-shelf LLM (e.g., ChatGPT, Llama, etc.) with a simple instruction , like "write a document that answers the question" , to convert the user query into a generated fake document. This transformation of a short user question into a longer hypothetical text passage is the central idea of HyDE.

This generated fake document will most likely contain hallucinated numbers and false statements. However, this does not matter, because the fake document is encoded into an embedding vector by the encoder model and used for semantic similarity search.

> **Key insight:** According to the HyDE paper, the encoding model acts as a lossy compressor that filters out the hallucinated details of the generated fake document. This leaves a vector embedding that should be very similar to the embeddings of the corpus of real documents.

Finally, the retriever uses the generated fake document to search for the closest real documents in the document embedding space, usually via dot product or cosine similarity.

> **Summary:** Instead of performing a similarity search in the query-document embedding space, HyDE performs a similarity search in the (hypothetical) document-(real) document embedding space.

### Contriever

A natural question is: what is a contriever, and why does HyDE use one?

The HyDE paper is strongly motivated by the fact that a large enough dataset is not always available to train a retriever for query-document similarity search.

> **Definition:** A **contriever** is a retriever (embedding model) trained with contrastive learning.

**Contrastive learning** is a form of self-supervised learning where no labels are required for the training dataset. This is particularly useful when large amounts of labeled data are not available, such as when trying to train a retriever in a language other than English.

An embedding model trained with contrastive learning tries to distinguish between semantically similar text (high score) and semantically dissimilar text (low score).

#### How Contrastive Training Works

During the contrastive training process:

- Text pairs are selected either from the same document (**positive pair**) or from different documents (**negative pair**).
- The retriever is trained to distinguish between positive and negative document pairs , assigning high scores to positive documents and low scores to negative documents.

The trained contriever can then be used as-is, or as a pre-trained model for further fine-tuning with labeled data. The contriever is trained in a self-supervised manner by searching for similarities between documents , no labeled data is required. The HyDE instruction transforms user questions into this document space by creating fake documents.

## When to Use HyDE

The choice of embedding model is critical to understanding when HyDE will actually improve RAG retrieval.

### Case 1: General-Purpose Contrastive Models

A popular general-purpose and free-to-use encoder model is the **all-MiniLM-L12-v2** model from the sentence-transformers package, hosted on Hugging Face. According to its model card, this project trains sentence embedding models on very large sentence-level datasets using a self-supervised contrastive learning objective, using the pretrained microsoft/MiniLM-L12-H384-uncased model fine-tuned on a 1B sentence pairs dataset.

> **Key point:** This means the encoder model is exactly what HyDE was made for , it was trained without labels using self-supervised contrastive learning on document-document data pairs. HyDE should be able to improve retrieval performance for this type of embedding model.

### Case 2: Models Trained for Asymmetric Semantic Search

HyDE is not necessary if the encoder model has already been specifically trained in a supervised manner for semantic search , especially **asymmetric semantic search**.

> **Definition:** Asymmetric semantic search means having a short question and looking for a longer paragraph to answer that question , exactly what RAG is typically used for.

A popular training dataset for this type of encoder model is the **MS MARCO dataset**, originally a question-answering dataset containing real Bing questions and human-generated answers. Encoder models from the Sentence Transformers library, such as the "msmarco-_" models and the "multi-qa-_" models, are already trained on labeled question-document data and therefore should (in theory) not benefit from using HyDE.

### Case 3: Commercial Embedding Models

For most commercial embedding models, such as the text-embedding models from OpenAI, it is not publicly known how they were trained, so HyDE may or may not improve retrieval performance.

## Implementing HyDE

Below is a basic implementation of HyDE in Python.

### Step 1: Setting Up the LLM

A simple LLM class initializes a local Qwen2.5-0.5B-Instruct model. The model is small enough to run on the CPU if no GPU is available locally.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

class LLM:
    def __init__(
        self,
        model_name="Qwen/Qwen2.5-0.5B-Instruct",
    ):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.model = AutoModelForCausalLM.from_pretrained(
            model_name,
            torch_dtype="auto",
        ).to(self.device)
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

    def generate(self, prompt, temperature=0.7, max_new_tokens=256):
        messages = [{"role": "user", "content": prompt}]
        text = self.tokenizer.apply_chat_template(
            messages, tokenize=False, add_generation_prompt=True
        )
        model_inputs = self.tokenizer([text], return_tensors="pt").to(self.device)
        generated_ids = self.model.generate(
            **model_inputs,
            max_new_tokens=max_new_tokens,
            do_sample=True,
            temperature=temperature,
        )
        generated_ids = [
            output_ids[len(input_ids):]
            for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
        ]
        return self.tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
```

### Step 2: Setting Up the Encoder Model

An encoder model is needed to compute sentence embeddings. A local `all-MiniLM-L12-v2` contriever model can be loaded using the `sentence_transformers` library:

```python
from sentence_transformers import SentenceTransformer

encoder_model = SentenceTransformer("all-MiniLM-L12-v2", device="cpu")
```

### Step 3: Generating a Hypothetical Document

With the LLM and encoder model set up, hypothetical document encodings can be computed:

```python
qwen = LLM()
question = "was ronald reagon a democrat?"

hypothetical_document = qwen.generate(
    f"Write a paragraph that answers the question. Question: {question}"
)

print(hypothetical_document)
```

**Example output (hypothetical document):**

> Ronald Reagan, born on November 6, 1924, in Grand Prairie, Texas, was a renowned American politician who served as the 36th President of the United States from 1981 to 1989. As the first Republican candidate for president and the first sitting president since Richard Nixon's resignation, Reagan faced significant challenges during his tenure. Reagan's political career began with a successful run for office against the Democratic nominee Hubert Humphrey in the 1968 presidential election. However, he quickly became disillusioned with the Democratic Party's stance on civil rights and social justice...

> **Note:** At first glance, this looks like it could be from Wikipedia. However, it contains many hallucinated facts (e.g., incorrect birth date and location). Since this is not a real document, it is acceptable to have errors , the goal is only to approximate the embedding space of a real answer document.

### Step 4: Comparing Embeddings

Next, a real passage of text from Wikipedia is used to compute the embeddings of the question, the Wikipedia document, and the hypothetical document:

```python
wikipedia = """Ronald Wilson Reagan[a] (February 6, 1911 – June 5, 2004) was an American ...
A member of the Republican Party, he became an important figure in the American conservat..."""

hypothetical_document_embedding = encoder_model.encode(hypothetical_document)
question_embedding = encoder_model.encode(question)
wikipedia_embedding = encoder_model.encode(wikipedia)
```

Now it can be checked whether the hypothetical document embedding is actually closer to the real document embedding than the question embedding, using the encoder model's built-in similarity function (based on cosine similarity).

> **Definition:** Cosine similarity ranges from -1 to +1, where -1 means the embedding vectors point in opposite directions, 0 means they are exactly perpendicular, and +1 means they point in the same direction.

```python
print(encoder_model.similarity(hypothetical_document_embedding, wikipedia_embedding))
# tensor([[0.8039]])

print(encoder_model.similarity(question_embedding, wikipedia_embedding))
# tensor([[0.4566]])
```

> **Key result:** The hypothetical document embedding (0.8039) is much closer to the real document embedding than the question embedding (0.4566). Thus, HyDE has successfully reduced the domain gap between question and document.

This confirms that HyDE performs similarity search in the (fake) document-(real) document embedding space, rather than the question-document embedding space.

### The Trade-Off

It also took some additional computation to generate the hypothetical document using the LLM. This is the disadvantage of using HyDE , it adds an extra LLM call before the retrieval step.

## Is Implementing HyDE Worth It?

A study called "Searching for Best Practices in Retrieval-Augmented Generation" looked at different retrieval methods for RAG and found that HyDE improved retrieval performance compared to the baseline embedding model.

Key findings from the study:

- The combination of **hybrid search with HyDE** produced the best overall results.
- Concatenating the original query with the hypothetical document produced even better results.
- On the other hand, HyDE increases latency and cost by requiring additional LLM calls to transform each query into a fake document.

> **Recommendation:** Considering the best performance and tolerated latency, Hybrid Search with HyDE is recommended as the default retrieval method. Taking efficiency into consideration, Hybrid Search combines sparse retrieval (BM25) and dense retrieval (original embedding) and achieves notable performance with relatively low latency.

## Conclusion

HyDE is an advanced technique for improving the retrieval part of a RAG pipeline. By creating hypothetical fake documents from a query, it becomes possible to perform similarity search in the document-document embedding space, instead of the question-document embedding space.

HyDE has been proposed for use cases where the embedding model is not already fine-tuned for semantic search with labeled question-document data. Since HyDE requires only a few additional LLM calls, it is very easy to implement.

HyDE is a building block in a RAG toolkit that can be combined with other advanced RAG techniques, such as hybrid search and using a reranker after the retrieval step.
