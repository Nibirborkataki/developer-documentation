
# Chapter 8 — Complete RAG Pipeline

## 8.1 Overview

Chapters 5–7 built the individual components required for document retrieval:

```text
PDF
 ↓
Text Extraction
 ↓
Chunking
 ↓
Embeddings
 ↓
ChromaDB
```

However, storing document embeddings alone does not create a complete RAG application.

We now connect the retrieval system with Qwen3.

The final RAG pipeline becomes:

```text
User Question
 ↓
Question Embedding
 ↓
ChromaDB Similarity Search
 ↓
Relevant Chunks
 ↓
Context Construction
 ↓
Qwen3
 ↓
Grounded Answer
```

RAG stands for:

> Retrieval-Augmented Generation

---

# 8.2 Before Chapter 8

Before implementing RAG, our chat endpoint essentially worked like:

```text
Question
 ↓
Qwen3
 ↓
Answer
```

This is a normal LLM interaction.

The model is not required to use our PDF.

Therefore, it could answer questions based on its general model knowledge.

That is not what we want.

---

# 8.3 After Chapter 8

The new architecture is:

```text
Question
 ↓
Generate Question Embedding
 ↓
Search ChromaDB
 ↓
Retrieve Relevant Chunks
 ↓
Build Context
 ↓
Send Context + Question to Qwen3
 ↓
Generate Answer
```

Now Qwen3 receives information retrieved from our own documents.

---

# 8.4 What Is RAG?

RAG has two major stages.

## Retrieval

Find relevant information.

```text
Question
 ↓
Embedding
 ↓
Vector Search
 ↓
Relevant Chunks
```

## Generation

Use the retrieved information to generate the answer.

```text
Question
+
Retrieved Context
 ↓
Qwen3
 ↓
Answer
```

Therefore:

```text
RAG = Retrieval + Generation
```

---

# 8.5 Complete Architecture

```text
                           LocalRAG AI
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
       Document Pipeline                     Chat Pipeline
              │                                   │
             PDF                              Question
              ↓                                   ↓
          PyMuPDF                           Embedding
              ↓                                   ↓
          Cleaning                         ChromaDB Search
              ↓                                   ↓
          Chunking                       Relevant Chunks
              ↓                                   ↓
        Embeddings                        Build Context
              ↓                                   ↓
          ChromaDB                            Qwen3
                                                  ↓
                                               Answer
```

The two pipelines meet at ChromaDB.

---

# 8.6 Step 1 — Retrieval Service

We created:

```text
app/services/retrieval_service.py
```

Its responsibility is:

> Convert a question into an embedding and retrieve relevant chunks from ChromaDB.

Implementation:

```python
from app.database.chroma_service import collection
from app.services.embedding_service import generate_embedding


def retrieve_chunks(
    question: str,
    n_results: int = 5
) -> list[dict]:

    question_embedding = generate_embedding(
        question
    )

    results = collection.query(
        query_embeddings=[question_embedding],
        n_results=n_results
    )

    retrieved_chunks = []

    documents = results["documents"][0]
    metadatas = results["metadatas"][0]
    distances = results["distances"][0]

    for document, metadata, distance in zip(
        documents,
        metadatas,
        distances
    ):

        retrieved_chunks.append({
            "text": document,
            "metadata": metadata,
            "distance": distance
        })

    return retrieved_chunks
```

---

# 8.7 Retrieval Process

When the application receives:

```text
What is the name of the fund manager?
```

the question first goes to the embedding model.

```text
Question
 ↓
nomic-embed-text
 ↓
Question Vector
```

Then:

```text
Question Vector
 ↓
ChromaDB
 ↓
Similarity Search
```

ChromaDB returns the most relevant chunks.

---

# 8.8 Why Use Question Embeddings?

The stored document chunks already have embeddings.

For example:

```text
Document Chunk
      ↓
Embedding
      ↓
Vector A
```

When the user asks a question:

```text
Question
      ↓
Embedding
      ↓
Vector B
```

ChromaDB compares the question vector against stored vectors.

Conceptually:

```text
Vector B
   ↓
Compare with stored vectors
   ↓
Find closest vectors
   ↓
Return relevant chunks
```

This enables semantic search.

---

# 8.9 `n_results`

The retrieval function uses:

```python
n_results=5
```

This means the system requests the top five relevant chunks.

For example:

```text
Question
 ↓
ChromaDB
 ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
Chunk 5
```

The number can later be tuned.

Too few chunks may miss important information.

Too many chunks may introduce irrelevant information.

This will be important in Chapter 9.

---

# 8.10 Retrieval Testing

We created:

```text
tests/test_retrieval.py
```

The test asks:

```text
What does the scheme invest in?
```

and displays:

- retrieved chunk
    
- distance
    
- source file
    
- page number
    

Example:

```text
============================================================
RESULT 1
============================================================

Distance: ...

Source: test_factsheet.pdf
Page: 5

The scheme primarily invests...
```

This confirmed that ChromaDB was able to retrieve semantically relevant information.

---

# 8.11 What Is Distance?

ChromaDB returns a distance value for retrieved results.

Conceptually:

```text
Smaller distance
      ↓
More similar

Larger distance
      ↓
Less similar
```

For example:

```text
Chunk A → 0.15
Chunk B → 0.29
Chunk C → 0.71
```

Chunk A is generally more similar to the query than Chunk C.

The exact interpretation depends on the configured distance space.

The important point is that this value can later help us determine whether a result is sufficiently relevant.

---

# 8.12 Step 2 — Build the Context

ChromaDB returns multiple chunks.

Qwen3 needs those chunks as text.

Therefore we created:

```text
app/services/rag_service.py
```

The first function is:

```python
def build_context(chunks: list[dict]) -> str:

    context_parts = []

    for chunk in chunks:

        context_parts.append(
            chunk["text"]
        )

    return "\n\n".join(context_parts)
```

---

# 8.13 What Does `build_context()` Do?

Suppose ChromaDB returns:

```text
Chunk 1:
The scheme invests primarily in equity.

Chunk 2:
The scheme follows a growth strategy.

Chunk 3:
The fund is managed by Mr. Pranav Mise.
```

`build_context()` combines them:

```text
The scheme invests primarily in equity.

The scheme follows a growth strategy.

The fund is managed by Mr. Pranav Mise.
```

This becomes the context supplied to Qwen3.

---

# 8.14 Step 3 — Modify the Ollama Service

Previously, Qwen3 received only:

```text
Question
```

Now it receives:

```text
Question
+
Retrieved Context
```

The service uses:

```python
MODEL_NAME = "qwen3:4b"
```

and:

```python
response = ollama.chat(
    model=MODEL_NAME,
    messages=[
        {
            "role": "user",
            "content": prompt
        }
    ]
)
```

The prompt instructs the model to use the provided context.

---

# 8.15 RAG Prompt

The prompt is:

```python
prompt = f"""
You are a document-based AI assistant.

Answer the user's question using only the
information provided in the context.

If the answer cannot be found in the context,
say that the information is not available
in the provided documents.

Do not invent or assume information.

Context:
{context}

Question:
{question}
"""
```

This is an important part of grounding.

---

# 8.16 Why Tell Qwen3 to Use Only the Context?

Without this instruction, the LLM could use its general knowledge.

For example:

```text
User:
What is the fund manager?
```

The model might attempt to answer based on information learned during training.

We instead want:

```text
Document
 ↓
Retrieve evidence
 ↓
Qwen3
 ↓
Answer based on evidence
```

This makes the chatbot document-focused.

---

# 8.17 Hallucination Control — Initial Layer

The prompt includes:

```text
If the answer cannot be found in the context,
say that the information is not available
in the provided documents.

Do not invent or assume information.
```

This provides an initial level of hallucination control.

However, this is **not sufficient by itself**.

The model may still receive irrelevant chunks and attempt to construct an answer.

Therefore, Chapter 9 will introduce an actual relevance check.

---

# 8.18 Step 4 — Complete RAG Service

The complete service is:

```python
from app.services.retrieval_service import retrieve_chunks
from app.services.ollama_service import generate_answer


def build_context(chunks: list[dict]) -> str:

    context_parts = []

    for chunk in chunks:

        context_parts.append(
            chunk["text"]
        )

    return "\n\n".join(context_parts)


def answer_question(
    question: str
) -> dict:

    retrieved_chunks = retrieve_chunks(
        question,
        n_results=5
    )

    context = build_context(
        retrieved_chunks
    )

    answer = generate_answer(
        question,
        context
    )

    return {
        "answer": answer,
        "sources": [
            chunk["metadata"]
            for chunk in retrieved_chunks
        ]
    }
```

---

# 8.19 Complete RAG Flow

Calling:

```python
answer_question(
    "What is the name of the fund manager?"
)
```

causes:

```text
Question
   ↓
retrieve_chunks()
   ↓
Generate question embedding
   ↓
ChromaDB search
   ↓
Relevant chunks
   ↓
build_context()
   ↓
Context
   ↓
generate_answer()
   ↓
Qwen3
   ↓
Answer
```

---

# 8.20 Step 5 — Testing the Complete RAG Pipeline

We created:

```text
tests/test_rag.py
```

```python
from app.services.rag_service import answer_question


question = "What does the scheme invest in?"


result = answer_question(
    question
)


print("\nANSWER")
print("=" * 60)

print(result["answer"])


print("\nSOURCES")
print("=" * 60)

for source in result["sources"]:

    print(
        f"{source['file_name']} "
        f"- Page {source['page_number']} "
        f"- Chunk {source['chunk_index']}"
    )
```

---

# 8.21 Testing Result

The test successfully produced:

```text
ANSWER
============================================================
...
```

followed by:

```text
SOURCES
============================================================
test_factsheet.pdf - Page X - Chunk X
...
```

This confirmed that:

1. The question was embedded.
    
2. ChromaDB retrieved relevant chunks.
    
3. Context was constructed.
    
4. Qwen3 received the context.
    
5. Qwen3 generated an answer.
    
6. Source metadata was returned.
    

---

# 8.22 Step 6 — Integrating RAG with FastAPI

Before Chapter 8, the endpoint used the direct LLM service.

The new endpoint uses:

```python
answer_question(
    request.question
)
```

The endpoint becomes:

```python
@app.post("/api/chat")
def chat(request: ChatRequest):

    result = answer_question(
        request.question
    )

    return {
        "question": request.question,
        "answer": result["answer"],
        "sources": result["sources"]
    }
```

---

# 8.23 API Response

After integration, the API returns:

```json
{
  "question": "What the name of fund manager?",
  "answer": "The context mentions multiple fund managers...",
  "sources": [
    {
      "chunk_index": 7,
      "page_number": 9,
      "file_name": "test_factsheet.pdf"
    },
    {
      "file_name": "test_factsheet.pdf",
      "page_number": 21,
      "chunk_index": 6
    }
  ]
}
```

The exact answer and source ordering depend on the retrieved chunks.

---

# 8.24 What This Result Proves

This response demonstrates that the complete pipeline is functioning.

The API is not simply asking Qwen3:

```text
"What is the name of fund manager?"
```

Instead, it is doing:

```text
Question
 ↓
Vector Retrieval
 ↓
Document Context
 ↓
Qwen3
 ↓
Answer
```

This is the key milestone of the project.

---

# 8.25 An Observation From Testing

For the question:

```text
What the name of fund manager?
```

the model returned a relatively verbose response mentioning multiple fund managers.

It eventually identified:

```text
Mr. Pranav Mise
```

but it also discussed:

- Mr. Milan Mody
    
- Mr. Mayur Patel
    
- Mr. Rahul Khetawat
    

This shows an important limitation in the current implementation.

We currently retrieve:

```text
Top 5 chunks
```

without checking whether every retrieved chunk is sufficiently relevant.

Therefore, Qwen3 may receive unrelated information.

---

# 8.26 Why This Matters

Suppose the user asks:

```text
What is the fund manager?
```

but ChromaDB returns:

```text
Chunk 1 → Fund manager
Chunk 2 → Disclaimer
Chunk 3 → Another fund manager
Chunk 4 → Another scheme
Chunk 5 → General information
```

Qwen3 sees all five chunks.

It may combine them and produce an unnecessarily complicated answer.

This is not necessarily a failure of the LLM.

It is partly a **retrieval-quality problem**.

---

# 8.27 Why Chapter 9 Is Necessary

We need another layer:

```text
Question
 ↓
Retrieve chunks
 ↓
Check relevance
 ↓
Are results relevant?
 ├── NO → Refuse
 └── YES
       ↓
     Qwen3
       ↓
     Answer
```

For example:

```text
User:
Who is the Prime Minister of Japan?

 ↓

ChromaDB
 ↓

No sufficiently relevant document chunk

 ↓

"I don't have enough information
in the provided documents."
```

This is much safer than allowing Qwen3 to answer using general knowledge.

---

# 8.28 Chapter 8 Architecture

The final Chapter 8 flow is:

```text
                         User
                           │
                           ↓
                       Question
                           │
                           ↓
                  nomic-embed-text
                           │
                           ↓
                    Question Vector
                           │
                           ↓
                       ChromaDB
                           │
                           ↓
                   Relevant Chunks
                           │
                           ↓
                    Context Builder
                           │
                           ↓
                 Question + Context
                           │
                           ↓
                         Qwen3
                           │
                           ↓
                        Answer
                           │
                           ↓
                       FastAPI
                           │
                           ↓
                     React Frontend
```

---

# 8.29 Document Pipeline vs Chat Pipeline

It is important to distinguish the two pipelines.

## Document Pipeline

Runs when documents are ingested:

```text
PDF
 ↓
PyMuPDF
 ↓
Cleaning
 ↓
Chunking
 ↓
Embedding
 ↓
ChromaDB
```

## Chat Pipeline

Runs when the user asks a question:

```text
Question
 ↓
Embedding
 ↓
ChromaDB Search
 ↓
Relevant Chunks
 ↓
Context
 ↓
Qwen3
 ↓
Answer
```

The two pipelines meet at:

```text
ChromaDB
```

---

# 8.30 Current Backend Structure

After Chapter 8:

```text
backend/
│
├── app/
│   │
│   ├── main.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   └── chat.py
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── pdf_service.py
│   │   ├── chunking_service.py
│   │   ├── embedding_service.py
│   │   ├── retrieval_service.py
│   │   ├── rag_service.py
│   │   └── ollama_service.py
│   │
│   └── database/
│       ├── __init__.py
│       └── chroma_service.py
│
├── data/
│   ├── documents/
│   │   └── test_factsheet.pdf
│   │
│   └── chroma/
│
├── tests/
│   ├── test_pdf.py
│   ├── test_chunking.py
│   ├── test_embeddings.py
│   ├── test_chroma.py
│   ├── test_search.py
│   ├── test_ingestion.py
│   ├── test_retrieval.py
│   └── test_rag.py
│
├── .venv/
└── requirements.txt
```

---

# 8.31 Chapter 8 Testing Checklist

The following tests have been completed successfully:

-  PDF extraction
    
-  Text cleaning
    
-  Chunking
    
-  Chunk overlap
    
-  Metadata preservation
    
-  Embedding generation
    
-  ChromaDB storage
    
-  Vector search
    
-  Retrieval service
    
-  Context construction
    
-  Qwen3 generation
    
-  Complete RAG pipeline
    
-  FastAPI integration
    
-  Source metadata returned through API
    

---

# 8.32 Key Interview Concepts

## What does RAG stand for?

Retrieval-Augmented Generation.

## What are the two major stages?

```text
Retrieval
+
Generation
```

## Why use RAG instead of directly asking an LLM?

RAG allows the model to generate answers using external/private documents instead of relying only on information learned during training.

## What is the role of ChromaDB?

It stores document embeddings and performs vector similarity search.

## What is the role of the embedding model?

It converts text into vectors so semantic similarity can be calculated.

## What is the role of Qwen3?

It generates the final natural-language answer using the retrieved context.

## Why preserve metadata?

To identify the source document and page of retrieved information.

## Is the current system fully hallucination-proof?

No.

The current prompt provides basic grounding, but we still need relevance filtering and threshold-based refusal.

That is the focus of Chapter 9.

---

# 8.33 Chapter 8 Summary

We transformed the project from a simple local LLM application into a functioning RAG application.

Before:

```text
Question
 ↓
Qwen3
 ↓
Answer
```

After:

```text
Question
 ↓
Embedding
 ↓
ChromaDB
 ↓
Relevant Context
 ↓
Qwen3
 ↓
Grounded Answer
```

The system can now retrieve information from the indexed mutual fund factsheet and use that information to generate answers.

---

# 8.34 Chapter 8 Status

**Status: ✅ Complete**

The complete RAG pipeline is working successfully.

The next improvement is to make retrieval more reliable by determining whether retrieved chunks are actually relevant to the user's question.

Next:

```text
Chapter 9 — Relevance & Hallucination Control
```