
# Chapter 7 — Embeddings + ChromaDB

## 7.1 Overview

Chapter 6 produced clean document chunks.

The next step is to make those chunks searchable by **meaning**.

We do this using:

1. An embedding model
    
2. Vector embeddings
    
3. ChromaDB
    

The pipeline becomes:

```text
PDF
 ↓
PyMuPDF
 ↓
Text Cleaning
 ↓
Chunking
 ↓
Embedding Model
 ↓
Vector
 ↓
ChromaDB
```

For LocalRAG AI, we use:

```text
Embedding Model:
nomic-embed-text

Vector Database:
ChromaDB
```

---

# 7.2 What Is an Embedding?

An embedding is a numerical representation of text.

For example:

```text
"The scheme invests in equity securities."
```

is converted into a vector conceptually similar to:

```text
[
    0.021,
   -0.183,
    0.442,
    0.091,
   -0.317,
    ...
]
```

The actual vector contains many dimensions.

The important concept is:

> Text with similar meanings tends to produce vectors that are close to each other in vector space.

---

# 7.3 Why Do We Need Embeddings?

Suppose the document contains:

```text
The scheme primarily invests in equity and equity-related securities.
```

The user asks:

```text
Where does the fund put its money?
```

The exact words are different.

Keyword search might have difficulty connecting:

```text
invests
```

with:

```text
put its money
```

Embeddings allow us to search based on semantic meaning.

Therefore:

```text
Question
 ↓
Embedding
 ↓
Semantic Search
 ↓
Relevant Document Chunks
```

---

# 7.4 Embedding Model vs LLM

We use two different models.

### Qwen3

```text
Question + Context
        ↓
      Qwen3
        ↓
      Answer
```

### nomic-embed-text

```text
Text
 ↓
nomic-embed-text
 ↓
Vector
```

Therefore:

|Model|Purpose|
|---|---|
|`qwen3:4b`|Generate answers|
|`nomic-embed-text`|Generate embeddings|

They perform different jobs.

---

# 7.5 Local Embeddings

The embedding model runs through Ollama.

We already installed:

```powershell
ollama pull nomic-embed-text
```

Therefore:

```text
Application
    ↓
Ollama
    ↓
nomic-embed-text
    ↓
Embedding
```

No external AI API is required.

This matches the project's goal of keeping the RAG system local and private.

---

# 7.6 Testing the Embedding Model

Before integrating it into the application, we tested the model separately.

File:

```text
tests/test_embeddings.py
```

```python
import ollama


text = (
    "The scheme primarily invests in equity "
    "and equity related securities."
)


response = ollama.embed(
    model="nomic-embed-text",
    input=text
)


embedding = response["embeddings"][0]


print("Embedding generated successfully.")
print("Vector dimensions:", len(embedding))
print("First 10 values:", embedding[:10])
```

Run:

```powershell
python tests/test_embeddings.py
```

Expected output:

```text
Embedding generated successfully.
Vector dimensions: ...
First 10 values: [...]
```

The exact vector values depend on the embedding model.

---

# 7.7 What Happens Internally?

This:

```python
response = ollama.embed(
    model="nomic-embed-text",
    input=text
)
```

sends the text to the local Ollama embedding model.

The model processes the text and generates a vector representation.

Then:

```python
embedding = response["embeddings"][0]
```

extracts the generated vector.

Conceptually:

```text
Text
 ↓
Tokenizer / Model Processing
 ↓
Numerical Representation
 ↓
Embedding Vector
```

---

# 7.8 Embedding Service

Instead of placing embedding logic throughout the application, we created a reusable service.

File:

```text
app/services/embedding_service.py
```

```python
import ollama


EMBEDDING_MODEL = "nomic-embed-text"


def generate_embedding(text: str) -> list[float]:
    """
    Generate an embedding vector for the given text
    using the local Ollama embedding model.
    """

    response = ollama.embed(
        model=EMBEDDING_MODEL,
        input=text
    )

    return response["embeddings"][0]
```

Now other parts of the application can simply call:

```python
generate_embedding(text)
```

---

# 7.9 Why Use a Separate Service?

Our architecture separates responsibilities.

```text
main.py
 ↓
API logic

pdf_service.py
 ↓
PDF extraction

chunking_service.py
 ↓
Text chunking

embedding_service.py
 ↓
Embedding generation
```

This makes the application:

- easier to maintain
    
- easier to test
    
- easier to debug
    
- easier to extend
    

It also prevents `main.py` from becoming a large file containing all business logic.

---

# 7.10 What Is ChromaDB?

ChromaDB is a vector database.

Traditional databases store structured information such as:

```text
ID
Name
Age
Email
```

A vector database stores information such as:

```text
ID
Document
Embedding
Metadata
```

For LocalRAG AI, a record conceptually looks like:

```text
ID:
test_factsheet_page_5_chunk_2

Document:
"The scheme primarily invests..."

Embedding:
[0.021, -0.183, 0.442, ...]

Metadata:
{
    file_name: "test_factsheet.pdf",
    page_number: 5,
    chunk_index: 2
}
```

---

# 7.11 Why ChromaDB?

The project needs to answer questions such as:

```text
What is the expense ratio?
```

Instead of searching the entire PDF, we want:

```text
Question
 ↓
Question embedding
 ↓
Vector search
 ↓
Top relevant chunks
```

ChromaDB provides the vector storage and similarity-search functionality required for this.

---

# 7.12 ChromaDB Directory

Our persistent vector database is stored here:

```text
backend/
└── data/
    └── chroma/
```

The application initializes ChromaDB using:

```python
import chromadb


CHROMA_PATH = "data/chroma"


client = chromadb.PersistentClient(
    path=CHROMA_PATH
)
```

---

# 7.13 Why PersistentClient?

We want the embeddings to survive application restarts.

Without persistence:

```text
Start application
 ↓
Create vectors
 ↓
Stop application
 ↓
Vectors disappear
```

With persistence:

```text
Create vectors
 ↓
Store on disk
 ↓
Restart application
 ↓
Vectors remain available
```

This is why we use:

```python
chromadb.PersistentClient()
```

---

# 7.14 ChromaDB Collection

We create a collection:

```python
collection = client.get_or_create_collection(
    name="documents"
)
```

The collection is called:

```text
documents
```

A useful mental model is:

```text
ChromaDB
   │
   └── documents
          │
          ├── Chunk 1
          ├── Chunk 2
          ├── Chunk 3
          └── ...
```

A collection is conceptually similar to a table in a traditional database, although the underlying functionality is different.

---

# 7.15 Storing a Chunk

A document chunk contains:

```text
ID
Document
Embedding
Metadata
```

We store these using:

```python
collection.add(
    ids=ids,
    documents=documents,
    embeddings=embeddings,
    metadatas=metadatas
)
```

For example:

```python
collection.add(
    ids=["test_1"],
    documents=[
        "The scheme primarily invests in equity."
    ],
    embeddings=[embedding],
    metadatas=[
        {
            "file_name": "test_factsheet.pdf",
            "page_number": 5,
            "chunk_index": 0
        }
    ]
)
```

---

# 7.16 Metadata in ChromaDB

Metadata allows us to preserve document source information.

Example:

```python
{
    "file_name": "test_factsheet.pdf",
    "page_number": 5,
    "chunk_index": 0
}
```

Later, when the RAG system retrieves this chunk, we can use:

```text
file_name
page_number
chunk_index
```

to identify its origin.

This will eventually support source citations in the UI.

---

# 7.17 ChromaDB Service

File:

```text
app/database/chroma_service.py
```

Current implementation:

```python
import chromadb


CHROMA_PATH = "data/chroma"


client = chromadb.PersistentClient(
    path=CHROMA_PATH
)


collection = client.get_or_create_collection(
    name="documents"
)


def add_chunks(
    chunks: list[dict],
    embeddings: list[list[float]]
) -> None:
    """
    Store document chunks and their embeddings
    in ChromaDB.
    """

    ids = []

    documents = []

    metadatas = []

    for chunk in chunks:

        chunk_id = (
            f"{chunk['file_name']}"
            f"_page_{chunk['page_number']}"
            f"_chunk_{chunk['chunk_index']}"
        )

        ids.append(chunk_id)

        documents.append(chunk["text"])

        metadatas.append({
            "file_name": chunk["file_name"],
            "page_number": chunk["page_number"],
            "chunk_index": chunk["chunk_index"]
        })

    collection.add(
        ids=ids,
        documents=documents,
        embeddings=embeddings,
        metadatas=metadatas
    )
```

---

# 7.18 Why Create Unique IDs?

Each chunk needs an identifier.

We generate IDs such as:

```text
test_factsheet.pdf_page_5_chunk_0
```

or:

```text
test_factsheet.pdf_page_10_chunk_2
```

The structure is:

```text
file
+
page
+
chunk
```

This helps with:

- debugging
    
- duplicate detection
    
- document management
    
- updating documents
    
- deleting documents
    

---

# 7.19 First ChromaDB Test

Before processing the entire PDF, we tested ChromaDB using one chunk.

File:

```text
tests/test_chroma.py
```

The test performed:

```text
Text
 ↓
Embedding
 ↓
ChromaDB
```

The document was stored using:

```python
collection.add(...)
```

Successful output:

```text
Document stored successfully.
```

This confirmed that ChromaDB persistence and vector storage were working.

---

# 7.20 Similarity Search

After storing a document, we tested retrieval.

Stored document:

```text
The scheme primarily invests in equity and
equity related securities.
```

Question:

```text
What does the scheme invest in?
```

The question is converted into an embedding.

```text
Question
 ↓
nomic-embed-text
 ↓
Question Vector
```

ChromaDB then searches for vectors that are most similar.

```text
Question Vector
       ↓
   ChromaDB
       ↓
Similarity Search
       ↓
Relevant Chunk
```

---

# 7.21 Search Code

The basic search looks like:

```python
results = collection.query(
    query_embeddings=[question_embedding],
    n_results=1
)
```

`n_results=1` means we request the most relevant result.

Later, we will likely retrieve multiple chunks:

```python
n_results=5
```

because a question may require information from several parts of the document.

---

# 7.22 What Is Similarity?

Imagine:

```text
Vector A
[0.2, 0.4, 0.8]

Vector B
[0.3, 0.5, 0.7]
```

These vectors are relatively close.

Another vector:

```text
Vector C
[-0.9, 0.1, -0.7]
```

may be much farther away.

Vector databases use mathematical distance/similarity calculations to determine which vectors are most relevant.

One common concept is:

```text
Cosine Similarity
```

Conceptually:

```text
similarity =
cosine of the angle between vectors
```

The important point for this project is:

> ChromaDB performs the vector search for us.

We do not need to manually calculate similarity for every stored vector.

---

# 7.23 Complete Ingestion Pipeline

After testing individual components, we connect everything.

The complete pipeline becomes:

```text
PDF
 ↓
PyMuPDF
 ↓
Pages
 ↓
Chunking
 ↓
Chunks
 ↓
nomic-embed-text
 ↓
Embeddings
 ↓
ChromaDB
```

---

# 7.24 Ingestion Test

File:

```text
tests/test_ingestion.py
```

```python
from app.services.pdf_service import extract_text_from_pdf
from app.services.chunking_service import chunk_pages
from app.services.embedding_service import generate_embedding
from app.database.chroma_service import add_chunks


pdf_path = "data/documents/test_factsheet.pdf"


# Step 1: Extract PDF text
pages = extract_text_from_pdf(pdf_path)


# Step 2: Create chunks
chunks = chunk_pages(
    pages,
    chunk_size=1000,
    overlap=200
)


print(f"Pages extracted: {len(pages)}")
print(f"Chunks created: {len(chunks)}")


# Step 3: Generate embeddings
embeddings = []


for index, chunk in enumerate(chunks):

    print(
        f"Generating embedding "
        f"{index + 1}/{len(chunks)}"
    )

    embedding = generate_embedding(
        chunk["text"]
    )

    embeddings.append(embedding)


# Step 4: Store in ChromaDB
add_chunks(
    chunks,
    embeddings
)


print("Ingestion completed successfully.")
```

Run:

```powershell
python tests/test_ingestion.py
```

Expected flow:

```text
Pages extracted: 22
Chunks created: XX

Generating embedding 1/XX
Generating embedding 2/XX
Generating embedding 3/XX
...

Ingestion completed successfully.
```

The exact chunk count depends on the PDF and chunking configuration.

---

# 7.25 Issue — Duplicate IDs

One issue we need to understand is duplicate ingestion.

Suppose we run:

```powershell
python tests/test_ingestion.py
```

and then run it again.

The same IDs may already exist:

```text
test_factsheet.pdf_page_5_chunk_0
```

ChromaDB can reject the duplicate IDs.

This is expected behavior.

It protects the database from unintentionally creating duplicate records.

---

# 7.26 How We Handle Duplicate Data

During development, the simplest approach is to clear the development ChromaDB data when necessary.

However, we should **not** blindly delete the database every time the application starts.

The production design should eventually be:

```text
Upload PDF
 ↓
Identify document
 ↓
Check whether it already exists
 ↓
Update / replace if necessary
 ↓
Generate chunks
 ↓
Generate embeddings
 ↓
Store vectors
```

This will be implemented properly when we build document upload and management.

---

# 7.27 Why We Test Components Separately

We deliberately didn't immediately connect everything.

We tested:

```text
Test 1
Embedding model
```

then:

```text
Test 2
ChromaDB storage
```

then:

```text
Test 3
Similarity search
```

and finally:

```text
Test 4
Complete ingestion
```

This makes debugging much easier.

If the final ingestion fails, we can determine whether the problem is related to:

```text
PDF extraction
Chunking
Embedding
ChromaDB
```

rather than debugging the entire system as one large block.

---

# 7.28 Important Architecture Distinction

At this point, we have built the **document ingestion side** of RAG.

We have:

```text
PDF
 ↓
Chunks
 ↓
Embeddings
 ↓
ChromaDB
```

However, our `/api/chat` endpoint from Chapter 4 currently does this:

```text
Question
 ↓
Qwen3
 ↓
Answer
```

It does **not yet perform vector retrieval**.

Therefore, it is not yet a complete RAG chat pipeline.

The complete RAG pipeline will be built in Chapter 8:

```text
Question
 ↓
Question Embedding
 ↓
ChromaDB Search
 ↓
Relevant Chunks
 ↓
Prompt + Context
 ↓
Qwen3
 ↓
Grounded Answer
```

---

# 7.29 Chapter 7 Data Flow

### Document ingestion

```text
PDF
 ↓
PyMuPDF
 ↓
Pages
 ↓
Chunking
 ↓
Chunks
 ↓
nomic-embed-text
 ↓
Vectors
 ↓
ChromaDB
```

### Future question answering

```text
User Question
 ↓
nomic-embed-text
 ↓
Question Vector
 ↓
ChromaDB
 ↓
Relevant Chunks
 ↓
Qwen3
 ↓
Answer
```

---

# 7.30 Important Concepts for Interviews

### What is an embedding?

A numerical vector representation of text that captures semantic information.

### Why use embeddings?

To enable semantic similarity search rather than relying only on keyword matching.

### What is a vector database?

A database designed to store and search numerical vector representations efficiently.

### Why use ChromaDB?

It provides persistent vector storage and similarity search for our RAG application.

### What is the difference between an embedding model and an LLM?

The embedding model converts text into vectors, while the LLM generates natural-language responses.

### Why preserve metadata?

To identify the source document, page, and chunk associated with retrieved information.

### What is similarity search?

Finding stored vectors that are mathematically closest or most similar to a query vector.

### Why use the same embedding model for documents and questions?

The document chunks and user questions need to exist in the same vector space so their semantic similarity can be meaningfully compared.

---

# 7.31 Chapter 7 Summary

We introduced two major components:

```text
nomic-embed-text
        +
     ChromaDB
```

The embedding model converts:

```text
Text → Vector
```

ChromaDB stores:

```text
Vector
+
Document
+
Metadata
```

and performs:

```text
Question Vector
       ↓
Similarity Search
       ↓
Relevant Chunks
```

This creates the retrieval foundation of our RAG system.

---

# 7.32 Current Architecture

The project now looks like:

```text
                 LocalRAG AI
                      │
             Document Pipeline
                      │
                      ↓
                     PDF
                      │
                      ↓
                  PyMuPDF
                      │
                      ↓
                   Chunks
                      │
                      ↓
             nomic-embed-text
                      │
                      ↓
                  Embeddings
                      │
                      ↓
                  ChromaDB
```

The next chapter will connect the retrieval system to Qwen3.

---

# 7.33 Chapter 7 Status

**Status: 🔄 In Progress**

The individual embedding and ChromaDB components are implemented and tested.

The next step is to integrate retrieval with the existing `/api/chat` endpoint.

That will create the complete Retrieval-Augmented Generation pipeline.