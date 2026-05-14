# Vector Database

## The Core Problem

Traditional databases like SQL are built for **exact keyword matching** and struggle with understanding the *meaning* behind a query.

> **Example**: If you search for "car" in a SQL database, it won't return results about "automobile" or "vehicle" — even though they mean the same thing.

Vector databases solve this by enabling **semantic (meaning-based) matching** — finding content that is *conceptually similar*, not just textually identical.

---

## What are Vectors (Embeddings)?

Machine learning models convert data (text, images, code) into a list of floating-point numbers called **vectors** or **embeddings**. These numbers place content in a mathematical space where **similar meanings cluster together**.

```
"king"   → [0.25, 0.80, 0.11, ...]
"queen"  → [0.26, 0.79, 0.13, ...]   ← close to "king"
"banana" → [0.91, 0.02, 0.55, ...]   ← far from "king"
```

The closer two vectors are in this space, the more semantically similar the content is.

### Embedding Visualization

```
High-dimensional space (simplified to 2D):

        Science
           |
  Physics  |  Chemistry
     *      |      *
            |
 -----------+----------- Sports
            |
    Soccer  |  Basketball
       *    |    *
            |
```

Words/sentences with similar meanings end up near each other in this space.

---

## How a Vector Database Works

### Phase 1: Ingestion (Storing Data)

```
┌─────────────────────────────────────────────────────────┐
│                    INGESTION PIPELINE                   │
│                                                         │
│  Raw Data          Embedding Model       Vector Store   │
│  ─────────         ───────────────       ───────────    │
│  "The cat          ┌───────────┐         [0.12, 0.87,   │
│   sat on    ──────▶│  text-    │──────▶   0.45, 0.33,  │
│   the mat"         │embedding  │          ...]  + meta  │
│                    └───────────┘                        │
│                                                         │
│  ① Chunk text  ② Embed chunks  ③ Store vectors + meta  │
└─────────────────────────────────────────────────────────┘
```

**Steps:**

1. **Chunking** — Split raw documents into smaller pieces (paragraphs, sentences, etc.)
2. **Embedding Generation** — Pass each chunk through an embedding model (e.g., `text-embedding-3-small`) to get a vector
3. **Storage** — Store each vector alongside the original text and optional metadata (source, date, category)
4. **Indexing** — Build an ANN (Approximate Nearest Neighbor) index so searches stay fast even with millions of vectors

---

### Phase 2: Retrieval (Similarity Search)

```
┌─────────────────────────────────────────────────────────┐
│                    RETRIEVAL PIPELINE                   │
│                                                         │
│  User Query        Embedding Model       Vector Store   │
│  ──────────        ───────────────       ───────────    │
│  "What does        ┌───────────┐         Search top-K   │
│   a cat     ──────▶│  text-    │──────▶  similar vecs  │
│   eat?"            │embedding  │         ↓              │
│                    └───────────┘         Return chunks  │
│                                                         │
│  ① Embed query  ② Find nearest vectors  ③ Return top-K │
└─────────────────────────────────────────────────────────┘
```

**Steps:**

1. **Query Embedding** — The user's query is converted using the *same* embedding model used during ingestion
2. **Similarity Search** — The database finds vectors closest to the query vector (using cosine similarity or dot product)
3. **Post-processing** — Results are ranked by similarity score, metadata filters are applied, and top-K results are returned

---

## Similarity Metrics

| Metric | Best For | Formula |
|---|---|---|
| **Cosine Similarity** | Text / NLP | angle between vectors |
| **Dot Product** | Normalized vectors | `a · b` |
| **Euclidean Distance** | Dense numeric data | `√Σ(aᵢ - bᵢ)²` |

---

## Chunking Strategies

Chunking determines how you split your source documents before embedding. The strategy significantly impacts retrieval quality.

| Strategy | Description | Best For |
|---|---|---|
| **Fixed-size** | Split every N characters | Generic documents |
| **Sentence-based** | Split at sentence boundaries | Articles, books |
| **Line-by-line** | Each line is a chunk | Logs, transcripts, dialogues |
| **Semantic** | Split when topic changes | Long-form mixed content |
| **Recursive** | Try paragraphs → sentences → words | General-purpose RAG |

### Chunking Diagram

```
Original Document:
─────────────────────────────────────────
"Chapter 1: Cats are mysterious animals.
 They sleep 16 hours a day.
 Chapter 2: Dogs are loyal companions.
 They love to play fetch."
─────────────────────────────────────────

After Chunking (sentence-based):
┌──────────────────────────────────┐
│ Chunk 1: "Cats are mysterious    │
│  animals."                       │
├──────────────────────────────────┤
│ Chunk 2: "They sleep 16 hours    │
│  a day."                         │
├──────────────────────────────────┤
│ Chunk 3: "Dogs are loyal         │
│  companions."                    │
├──────────────────────────────────┤
│ Chunk 4: "They love to play      │
│  fetch."                         │
└──────────────────────────────────┘
```

---

## RAG (Retrieval-Augmented Generation)

RAG is the most common use case for vector databases. It enhances LLM accuracy by **retrieving relevant context** from an external knowledge base before generating a response.

### RAG Architecture

```
                        ┌─────────────────┐
                        │  Knowledge Base │
                        │  (your docs,    │
                        │   PDFs, DBs)    │
                        └────────┬────────┘
                                 │ Ingest & Embed
                                 ▼
User Query ──────────▶  ┌────────────────┐
                        │ Vector Database │
                        └───────┬────────┘
                                │ Top-K similar chunks
                                ▼
                        ┌────────────────┐
                        │  LLM Prompt    │
                        │                │
                        │ Context: ...   │
                        │ Question: ...  │
                        └───────┬────────┘
                                │
                                ▼
                        ┌────────────────┐
                        │    Answer      │
                        └────────────────┘
```

---

## Code Examples

### Example 1: Build a RAG Pipeline with LangChain + ChromaDB

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.documents import Document

# 1. Prepare your documents
docs = [
    Document(page_content="Cats are independent and curious animals. They sleep up to 16 hours a day."),
    Document(page_content="Dogs are social animals that love companionship. They need daily exercise."),
    Document(page_content="Parrots can mimic human speech and are highly intelligent birds."),
]

# 2. Split into chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=100, chunk_overlap=20)
chunks = splitter.split_documents(docs)

# 3. Create embeddings and store in ChromaDB
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Query the vector store
query = "Which animal sleeps a lot?"
results = vectorstore.similarity_search(query, k=2)

for r in results:
    print(r.page_content)
# Output: "Cats are independent and curious animals. They sleep up to 16 hours a day."
```

---

### Example 2: Semantic Similarity Search (No LLM needed)

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

sentences = [
    "The stock market crashed today.",
    "Equity markets experienced a sharp decline.",
    "I love eating pizza on Fridays.",
    "My cat knocked over my coffee.",
]

embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_texts(sentences, embeddings)

# Search for semantically similar sentences
results = vectorstore.similarity_search_with_score("Financial markets fell sharply", k=2)

for doc, score in results:
    print(f"Score: {score:.3f} | Text: {doc.page_content}")

# Output:
# Score: 0.921 | Text: Equity markets experienced a sharp decline.
# Score: 0.874 | Text: The stock market crashed today.
```

---

### Example 3: Full RAG Chain with LangChain

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.chains import RetrievalQA
from langchain_core.documents import Document

# Setup documents
docs = [
    Document(page_content="Our return policy allows returns within 30 days with a receipt."),
    Document(page_content="Shipping typically takes 5-7 business days for standard delivery."),
    Document(page_content="Premium members get free 2-day shipping on all orders over $50."),
]

# Build vector store
vectorstore = Chroma.from_documents(docs, OpenAIEmbeddings())

# Build RAG chain
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="claude-sonnet-4-6"),
    retriever=vectorstore.as_retriever(search_kwargs={"k": 2}),
)

# Ask a question
response = qa_chain.invoke({"query": "How long do I have to return something?"})
print(response["result"])
# Output: "You have 30 days to return items, as long as you have a receipt."
```

---

### Example 4: Metadata Filtering

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

docs = [
    Document(page_content="Python is great for data science.", metadata={"category": "programming"}),
    Document(page_content="The Eiffel Tower is in Paris.", metadata={"category": "geography"}),
    Document(page_content="JavaScript powers the modern web.", metadata={"category": "programming"}),
]

vectorstore = Chroma.from_documents(docs, OpenAIEmbeddings())

# Filter results by metadata
results = vectorstore.similarity_search(
    "popular coding languages",
    k=2,
    filter={"category": "programming"}  # only search programming docs
)

for r in results:
    print(r.page_content)
# Output: Python and JavaScript results only (geography filtered out)
```

---

## Popular Vector Databases

| Database | Type | Best For |
|---|---|---|
| **ChromaDB** | Open-source, local | Prototyping, small projects |
| **FAISS** | Open-source, in-memory | Fast local search (Meta) |
| **Pinecone** | Managed cloud | Production at scale |
| **Weaviate** | Open-source / Cloud | Hybrid search + GraphQL |
| **Qdrant** | Open-source / Cloud | High-performance filtering |
| **pgvector** | PostgreSQL extension | Teams already using Postgres |

---

## Real-World Use Cases

| Domain | Use Case | Example |
|---|---|---|
| **Music / Video** | Recommendations | Spotify, Netflix — "songs like this" |
| **E-commerce** | Visual search | Pinterest — find similar images |
| **Customer Support** | FAQ search | Chatbots finding relevant help articles |
| **Cybersecurity** | Anomaly detection | Finding unusual log patterns |
| **Healthcare** | Document retrieval | Matching patient symptoms to research papers |
| **Code Search** | Semantic code search | GitHub Copilot finding relevant functions |

---

## ANN Indexing Algorithms

To avoid slow linear scans through millions of vectors, vector databases use **Approximate Nearest Neighbor (ANN)** indexes:

```
Linear Scan (slow):          ANN Index (fast):
Check vector 1 ──▶ match?    Build a tree/graph structure
Check vector 2 ──▶ match?    Navigate directly to
Check vector 3 ──▶ match?    approximate neighbors
...  (millions of checks)    (logarithmic time)
```

| Algorithm | Description |
|---|---|
| **HNSW** | Hierarchical Navigable Small World — most popular, used in Chroma/Weaviate |
| **IVF** | Inverted File Index — FAISS default, clusters vectors first |
| **ScaNN** | Google's algorithm — optimized for dot-product similarity |
