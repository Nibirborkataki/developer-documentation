

# Chapter 2 — Environment & AI Model Setup

## 2.1 Objective

The objective of this chapter is to prepare the local development environment required for LocalRAG AI.

The project is designed to run AI models locally instead of using external APIs.

---

## 2.2 System Architecture

The local AI environment consists of:

```text
Python Application
       ↓
Ollama
       ↓
Local AI Models
       ├── Qwen3 4B
       └── nomic-embed-text
````

---

## 2.3 Project Directory

The project is located at:

```
D:\AI-Projects\LocalRAG-AI
```

Initial structure:

```
LocalRAG-AI/
│
├── backend/
│
└── frontend/
```

The backend contains the Python/FastAPI application.

The frontend will contain the React application.

---

## 2.4 Why Separate Frontend and Backend?

Separating frontend and backend creates a clean architecture.

```
frontend/
    React UI

backend/
    FastAPI API
    RAG pipeline
    AI processing
```

Communication will happen through HTTP APIs.

```
React
  ↓ HTTP
FastAPI
  ↓
AI / RAG
```

---

## 2.5 Python Virtual Environment

A Python virtual environment was created inside the backend:

```
backend/
└── .venv/
```

Create a virtual environment:

```
python -m venv .venv
```

Activate it:

```
.\.venv\Scripts\Activate.ps1
```

The terminal should show:

```
(.venv) PS D:\AI-Projects\LocalRAG-AI\backend>
```

---

## 2.6 Why Use a Virtual Environment?

A virtual environment isolates project dependencies.

Without a virtual environment:

```
Project A
   ↓
Global Python packages

Project B
   ↓
Same global packages
```

Different projects can require different versions.

With virtual environments:

```
Project A
   ↓
Environment A

Project B
   ↓
Environment B
```

This prevents dependency conflicts.

---

## 2.7 Required Python Packages

The backend uses:

```
fastapi
uvicorn
ollama
python-dotenv
pymupdf
chromadb
```

Install them using:

```
pip install fastapi uvicorn ollama python-dotenv pymupdf chromadb
```

Save dependencies:

```
pip freeze > requirements.txt
```

---

# 2.8 Ollama

Ollama is used as the local LLM runtime.

Instead of calling an external API:

```
Python
 ↓
OpenAI/Other API
 ↓
Internet
```

we use:

```
Python
 ↓
Ollama
 ↓
Local Model
```

---

## 2.9 Installing Ollama

Ollama was installed on Windows using PowerShell:

```
cd D:\
irm https://ollama.com/install.ps1 | iex
```

Verify installation:

```
ollama --version
```

Installed version during development:

```
0.33.2
```

---

# 2.10 Model Storage

The C: drive had limited free space, so Ollama model storage was moved to the D: drive.

Created:

```
D:\AI-Models\Ollama
```

The environment variable was configured:

```
[Environment]::SetEnvironmentVariable(
    "OLLAMA_MODELS",
    "D:\AI-Models\Ollama",
    "User"
)
```

Verify:

```
[Environment]::GetEnvironmentVariable(
    "OLLAMA_MODELS",
    "User"
)
```

Expected:

```
D:\AI-Models\Ollama
```

---

## 2.11 Why Use OLLAMA_MODELS?

Ollama stores downloaded models separately from the current project directory.

The location from which the command is executed does not determine where models are stored.

The environment variable controls the model storage location.

---

# 2.12 Qwen3

The generation model used by the project is:

```
qwen3:4b
```

Download:

```
ollama pull qwen3:4b
```

The model is approximately 2.5 GB in size.

---

## 2.13 Embedding Model

The embedding model used is:

```
nomic-embed-text
```

Download:

```
ollama pull nomic-embed-text
```

Its purpose is different from Qwen3.

### Qwen3

Used for:

```
Question + Context
       ↓
Answer
```

### nomic-embed-text

Used for:

```
Text
 ↓
Embedding vector
```

---

## 2.14 Verify Models

Run:

```
ollama list
```

Expected models:

```
nomic-embed-text:latest
qwen3:4b
```

---

## 2.15 Model Responsibilities

|Model|Purpose|
|---|---|
|Qwen3 4B|Answer generation|
|nomic-embed-text|Text embeddings|

This separation is important in a RAG architecture.

---

## 2.16 Current Environment

At the end of this chapter:

```
Windows
   │
   ├── Python
   │
   ├── Virtual Environment
   │
   ├── FastAPI
   │
   ├── Uvicorn
   │
   └── Ollama
          │
          ├── qwen3:4b
          └── nomic-embed-text
```

The local AI environment is ready.

---

## 2.17 Verification Checklist

- [x]  Python installed
- [x]  Virtual environment created
- [x]  FastAPI installed
- [x]  Uvicorn installed
- [x]  Ollama installed
- [x]  Ollama model directory moved to D:
- [x]  Qwen3 downloaded
- [x]  Embedding model downloaded
- [x]  Models verified using `ollama list`

---

## 2.18 Interview Concepts

### What is Ollama?

Ollama is a tool/runtime that allows local execution and management of language models.

### Why use Ollama?

It allows applications to communicate with locally running AI models without requiring an external AI API.

### What is a local LLM?

A language model running on the user's own machine rather than on a remote AI service.

### What is an embedding model?

A model that converts text into numerical vectors representing semantic information.

### Why use two models?

Generation and embedding are different tasks.

Qwen3 generates answers, while nomic-embed-text generates embeddings for semantic search.