# Chapter 1 — Project Overview

> [!info] Project
> **LocalRAG AI — Private Local AI Knowledge Assistant**

---

## 1.1 Introduction

LocalRAG AI is a private, locally running AI-powered document assistant.

The primary purpose of the application is to allow users to upload documents such as PDF files and ask natural-language questions about their contents.

Instead of sending documents or questions to an external AI API, the application runs the AI model locally using **Ollama**.

The application is designed around the **Retrieval-Augmented Generation (RAG)** architecture.

The overall goal is:

```text
User
 ↓
Upload PDF
 ↓
Extract document text
 ↓
Split text into chunks
 ↓
Generate embeddings
 ↓
Store embeddings in vector database
 ↓
User asks a question
 ↓
Search relevant document chunks
 ↓
Send relevant context to local LLM
 ↓
Generate grounded answer
 ↓
Return answer + source information

```

## 1.2 Why This Project?

Traditional AI chatbots can answer general questions, but they do not automatically know the contents of a user's private documents.

For example, suppose a user uploads a mutual fund factsheet.

The user may ask:

- What is the AUM?
- Who is the fund manager?
- What is the expense ratio?
- What are the top holdings?
- What is the investment objective?
- What is the portfolio allocation?

The application should answer these questions using the uploaded document.

The key requirement is that the AI should not simply rely on its general knowledge.

It should retrieve information from the uploaded document and use that information to generate the answer.

---

## 1.3 Main Objectives

The project has the following objectives:

### Privacy

Documents remain on the user's local machine.

No external AI API is required.

### Local AI

The language model runs locally through Ollama.

### Document Question Answering

Users can ask questions about uploaded documents.

### Semantic Retrieval

The application should understand the meaning of a question rather than relying only on exact keyword matching.

### Source Attribution

Answers should eventually contain document and page information.

Example:

```
Answer:
The expense ratio is 0.85%.

Source:
ABC_Factsheet.pdf — Page 4
```

### Relevance Control

The application should avoid answering questions when the uploaded documents do not contain enough information.

Example:

```
User:
Who is the Prime Minister of Canada?

Assistant:
Sorry, I don't have enough information in the uploaded documents to answer that question.
```

---

## 1.4 Technology Stack

The project uses the following technologies:

|Layer|Technology|
|---|---|
|Frontend|React|
|Frontend tooling|Vite|
|Styling|Tailwind CSS|
|Backend|Python|
|API Framework|FastAPI|
|ASGI Server|Uvicorn|
|LLM Runtime|Ollama|
|Generation Model|Qwen3 4B|
|Embedding Model|nomic-embed-text|
|Vector Database|ChromaDB|
|PDF Processing|PyMuPDF|
|Metadata Database|SQLite|
|Version Control|Git|

---

## 1.5 Why Local AI?

A traditional application might communicate with an external AI provider:

```
Application
     ↓
Internet
     ↓
External AI API
     ↓
LLM
```

This introduces several concerns:

- API costs
- Internet dependency
- Data privacy
- API keys
- External service availability

LocalRAG AI instead uses:

```
Application
     ↓
Local Ollama Server
     ↓
Local Qwen3 Model
```

This means the core AI processing can happen entirely on the local machine.

---

## 1.6 High-Level Architecture

The application will eventually follow this architecture:

```
                    LocalRAG AI
                         │
                         ▼
                  React Frontend
                         │
                     HTTP/JSON
                         │
                         ▼
                  FastAPI Backend
                         │
            ┌────────────┴────────────┐
            │                         │
            ▼                         ▼
     Document Pipeline          Chat Pipeline
            │                         │
            ▼                         ▼
        PyMuPDF                  User Question
            │                         │
            ▼                         ▼
        Chunking                  Embedding
            │                         │
            ▼                         ▼
      Embedding Model            ChromaDB
            │                         │
            ▼                         ▼
        ChromaDB                Relevant Chunks
                                      │
                                      ▼
                                Context Builder
                                      │
                                      ▼
                                   Ollama
                                      │
                                      ▼
                                   Qwen3
                                      │
                                      ▼
                                   Answer
```

---

## 1.7 RAG Architecture

RAG stands for:

> Retrieval-Augmented Generation

It consists of three major concepts.

### Retrieval

Find relevant information from the stored documents.

### Augmentation

Add the retrieved information to the prompt sent to the language model.

### Generation

The LLM generates the final answer using the retrieved context.

The simplified pipeline is:

```
Question
   ↓
Retrieval
   ↓
Relevant document chunks
   ↓
Context
   ↓
LLM
   ↓
Answer
```

---

## 1.8 Document Ingestion Pipeline

When a PDF is uploaded:

```
PDF
 ↓
PyMuPDF
 ↓
Text extraction
 ↓
Page metadata
 ↓
Chunking
 ↓
Embedding generation
 ↓
ChromaDB
```

This process is called **document ingestion**.

The purpose of ingestion is to prepare documents before users start asking questions.

---

## 1.9 Query Pipeline

When a user asks a question:

```
User Question
 ↓
Generate Query Embedding
 ↓
Search ChromaDB
 ↓
Retrieve Top-K Relevant Chunks
 ↓
Build Context
 ↓
Send Context + Question to Qwen3
 ↓
Generate Answer
```

---

## 1.10 Relevance Control

A major feature of the application is preventing unsupported answers.

The system will eventually use a similarity threshold.

```
Question
   ↓
Vector Search
   ↓
Similarity Score
   ↓
Is score above threshold?
   │
   ├── No → Refuse / insufficient information
   │
   └── Yes → Generate answer
```

This helps reduce hallucination.

---

## 1.11 Current Development Status

Completed:

- Project architecture
- Python environment
- Ollama installation
- Local model configuration
- Qwen3 model
- Embedding model
- FastAPI application
- Health endpoint
- Chat endpoint
- Python → Ollama → Qwen3 communication

Current stage:

> **FastAPI backend completed up to the basic chat endpoint.**

Next stage:

> **PDF processing and document ingestion.**

---

## 1.12 Future Features

Potential future improvements include:

- Multiple document support
- Document management
- Source citations
- Page references
- Conversation history
- PDF preview
- Document comparison
- Table extraction
- OCR support
- Image understanding
- Charts and visualizations
- Multimodal RAG
- Automatic mutual fund factsheet ingestion
- Authentication
- Deployment

---

## 1.13 Portfolio Value

This project demonstrates practical knowledge of:

- Python
- FastAPI
- REST APIs
- React
- LLMs
- Ollama
- Embeddings
- Vector databases
- Semantic search
- RAG
- Prompt engineering
- Document processing
- AI application architecture

The project is more than a simple chatbot because it implements a complete local document intelligence pipeline.