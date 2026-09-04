
# Chapter 4 — FastAPI Backend 
## 4.1 Objective
The objective of this chapter is to create the backend API for LocalRAG AI. The backend acts as the communication layer between the frontend and the AI/RAG system.

The architecture is: 
```
 React 
 ↓
HTTP Request
 ↓ 
FastAPI
 ↓ 
AI / RAG Services
 ↓ 
FastAPI
 ↓ 
HTTP Response
 ↓ 
React
```

# 4.2 Why FastAPI?

FastAPI is a Python web framework designed for building APIs.

It provides:

- Request handling
- JSON responses
- Request validation
- Type hints
- Automatic API documentation
- OpenAPI support
- High performance

It is well suited to an AI backend because the main application logic is written in Python.

---

# 4.3 FastAPI and Uvicorn

These two tools have different responsibilities.

### FastAPI

Defines the application and API routes.

```
FastAPI
 ↓
Routes
 ↓
Business logic
```

### Uvicorn

Runs the FastAPI application as an ASGI server.

```
Browser
 ↓
HTTP
 ↓
Uvicorn
 ↓
FastAPI
```

---

# 4.4 Application Structure

Current backend structure:

```
backend/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   └── chat.py
│   │
│   └── services/
│       ├── __init__.py
│       └── ollama_service.py
│
├── data/
│
├── tests/
│
├── .venv/
│
└── requirements.txt
```

---

# 4.5 Creating the FastAPI Application

File:

```
app/main.py
```

Code:

```
from fastapi import FastAPI

from app.models.chat import ChatRequest
from app.services.ollama_service import generate_answer


app = FastAPI(
    title="LocalRAG AI",
    description="Private local AI knowledge assistant",
    version="1.0.0"
)


@app.get("/")
def root():
    return {
        "message": "Welcome to LocalRAG AI"
    }


@app.get("/api/health")
def health_check():
    return {
        "status": "healthy",
        "service": "LocalRAG AI API"
    }


@app.post("/api/chat")
def chat(request: ChatRequest):
    answer = generate_answer(request.question)

    return {
        "question": request.question,
        "answer": answer
    }
```

---

# 4.6 FastAPI Application Object

```
app = FastAPI(...)
```

This creates the FastAPI application instance.

The `app` object is responsible for registering routes and handling HTTP requests.

---

# 4.7 Application Metadata

```
app = FastAPI(
    title="LocalRAG AI",
    description="Private local AI knowledge assistant",
    version="1.0.0"
)
```

This metadata appears in the automatically generated API documentation.

---

# 4.8 GET Request

The first endpoint is:

```
@app.get("/")
def root():
    return {
        "message": "Welcome to LocalRAG AI"
    }
```

The route is:

```
GET /
```

When the client requests:

```
http://127.0.0.1:8000/
```

the function executes.

Response:

```
{
    "message": "Welcome to LocalRAG AI"
}
```

---

# 4.9 Health Endpoint

The second endpoint is:

```
@app.get("/api/health")
def health_check():
    return {
        "status": "healthy",
        "service": "LocalRAG AI API"
    }
```

Endpoint:

```
GET /api/health
```

Response:

```
{
    "status": "healthy",
    "service": "LocalRAG AI API"
}
```

---

# 4.10 Why Health Checks?

A health endpoint provides a simple way to determine whether the backend is running.

For example:

```
Frontend
   ↓
GET /api/health
   ↓
Backend
   ↓
healthy
```

It can later be used by deployment systems, monitoring systems, or the frontend.

---

# 4.11 Running FastAPI

The virtual environment should remain active.

```
uvicorn app.main:app --reload
```

The command can be understood as:

```
app.main
   ↓
main.py

:
   ↓

app
   ↓
FastAPI application object
```

`--reload` automatically restarts the development server when code changes.

---

# 4.12 Server Address

The development server runs at:

```
http://127.0.0.1:8000
```

`127.0.0.1` means the local machine.

Port:

```
8000
```

---

# 4.13 Automatic API Documentation

FastAPI automatically provides Swagger documentation.

Open:

```
http://127.0.0.1:8000/docs
```

The API can be tested directly from the browser.

FastAPI also provides ReDoc:

```
http://127.0.0.1:8000/redoc
```

---

# 4.14 Pydantic Request Model

File:

```
app/models/chat.py
```

Code:

```
from pydantic import BaseModel


class ChatRequest(BaseModel):
    question: str
```

This defines the expected structure of the chat request.

---

# 4.15 Why Pydantic?

Pydantic validates incoming data.

Our API expects:

```
{
    "question": "What is RAG?"
}
```

The model:

```
class ChatRequest(BaseModel):
    question: str
```

means:

```
question must exist
question should be a string
```

FastAPI uses the Pydantic model automatically.

---

# 4.16 POST Request

Our chat endpoint is:

```
@app.post("/api/chat")
def chat(request: ChatRequest):
```

The HTTP method is:

```
POST
```

The endpoint is:

```
/api/chat
```

---

# 4.17 Why POST?

POST is appropriate because the client is sending data to the server for processing.

Our request contains:

```
{
    "question": "What is RAG?"
}
```

The server processes this question and generates a response.

---

# 4.18 Request Body

The request body is:

```
{
    "question": "What is the decimal value of binary 101?"
}
```

FastAPI converts the JSON into:

```
ChatRequest(
    question="What is the decimal value of binary 101?"
)
```

We can then access the question using:

```
request.question
```

---

# 4.19 Query Parameters vs Request Body

A request like:

```
/api/chat?question=What%20is%20RAG
```

uses a query parameter.

However, our API expects a JSON request body:

```
{
    "question": "What is RAG?"
}
```

This distinction is important.

### Query parameter

Usually used for filtering or small parameters.

Example:

```
/api/documents?page=2
```

### Request body

Used for structured data sent to an API.

Example:

```
{
    "question": "What is RAG?",
    "document_id": "123"
}
```

---

# 4.20 Ollama Service

File:

```
app/services/ollama_service.py
```

Code:

```
import ollama


def generate_answer(question: str) -> str:

    response = ollama.chat(
        model="qwen3:4b",
        messages=[
            {
                "role": "user",
                "content": question
            }
        ]
    )

    return response["message"]["content"]
```

---

# 4.21 Why Create a Service Layer?

We don't want all application logic inside `main.py`.

Instead:

```
main.py
   ↓
API layer

ollama_service.py
   ↓
AI communication
```

This separates responsibilities.

Later the architecture will become:

```
API
 ↓
RAG Service
 ↓
Embedding Service
 ↓
Vector Database
 ↓
Ollama Service
```

---

# 4.22 Chat Request Flow

The complete current flow is:

```
Client
  │
  │ POST /api/chat
  │
  ▼
FastAPI
  │
  ▼
ChatRequest
  │
  ▼
generate_answer()
  │
  ▼
Ollama
  │
  ▼
Qwen3
  │
  ▼
Generated Answer
  │
  ▼
FastAPI JSON Response
```

---

# 4.23 Example Request

```
POST /api/chat
Content-Type: application/json
```

Body:

```
{
    "question": "What is the decimal value of binary 101?"
}
```

---

# 4.24 Example Response

```
{
    "question": "What is the decimal value of binary 101?",
    "answer": "The decimal value of binary 101 is 5."
}
```

The exact answer is generated by Qwen3.

---

# 4.25 Testing with Swagger

Open:

```
http://127.0.0.1:8000/docs
```

Find:

```
POST /api/chat
```

Click:

```
Try it out
```

Enter:

```
{
    "question": "What is the decimal value of binary 101?"
}
```

Click:

```
Execute
```

The API should return the generated response.

---

# 4.26 Testing the Health Endpoint

Open:

```
GET /api/health
```

Expected:

```
{
    "status": "healthy",
    "service": "LocalRAG AI API"
}
```

---

# 4.27 Error Encountered During Development

The following error occurred:

```
NameError: name 'generate_answer' is not defined
```

This happened because `generate_answer` was being called in `main.py` without being imported.

The required import is:

```
from app.services.ollama_service import generate_answer
```

This demonstrates an important Python concept:

> A function defined in another module must be imported before it can be used.

---

# 4.28 API Error Status Codes

The HTTP status code:

```
500 Internal Server Error
```

means the server encountered an unexpected error while processing the request.

For example:

```
Client
 ↓
POST /api/chat
 ↓
FastAPI
 ↓
Python exception
 ↓
500
```

Later, proper exception handling will be added so that users receive meaningful errors instead of generic server errors.

---

# 4.29 Current API Endpoints

At the end of Chapter 4:

|Method|Endpoint|Purpose|
|---|---|---|
|GET|`/`|API welcome message|
|GET|`/api/health`|Health check|
|POST|`/api/chat`|Send question to local Qwen3|

---

# 4.30 Future API Endpoints

The final application will eventually contain endpoints such as:

```
POST /api/chat
POST /api/documents/upload
GET /api/documents
DELETE /api/documents/{id}
```

Potential future endpoints:

```
GET /api/documents/{id}
GET /api/conversations
DELETE /api/conversations/{id}
```

---

# 4.31 Current Limitation

The current `/api/chat` endpoint sends the question directly to Qwen3.

Therefore:

```
Question
 ↓
Qwen3
 ↓
Answer
```

It does **not** yet retrieve information from PDFs.

This means it is currently a basic local AI chat endpoint rather than a RAG endpoint.

---

# 4.32 Target Architecture

The final `/api/chat` pipeline will become:

```
POST /api/chat
       │
       ▼
Question
       │
       ▼
Generate Query Embedding
       │
       ▼
Search ChromaDB
       │
       ▼
Retrieve Relevant Chunks
       │
       ▼
Check Relevance
       │
       ├── Not Relevant
       │       ↓
       │   Refuse Answer
       │
       └── Relevant
               ↓
        Build Context
               ↓
            Qwen3
               ↓
        Grounded Answer
               ↓
        Source Information
```

---

# 4.33 Interview Concepts

## What is an API?

An API is an interface through which different software components communicate.

In this project:

```
React
 ↓
FastAPI
```

---

## What is REST?

REST is an architectural style commonly used for web APIs.

Our application uses HTTP methods such as:

```
GET
POST
DELETE
```

---

## What is HTTP?

HTTP is the protocol used for communication between clients and servers.

Example:

```
Browser
 ↓ HTTP Request
FastAPI
 ↓ HTTP Response
Browser
```

---

## What is JSON?

JSON is a lightweight data format commonly used for API communication.

Example:

```
{
    "question": "What is RAG?"
}
```

---

## What is Pydantic?

Pydantic is used for data validation and parsing.

FastAPI uses Pydantic models to validate request data.

---

## What is Uvicorn?

Uvicorn is an ASGI server used to run FastAPI applications.

---

## What is CORS?

CORS stands for:

> Cross-Origin Resource Sharing

When our React frontend and FastAPI backend run on different origins, CORS configuration may be required.

Example:

```
React
http://localhost:5173

FastAPI
http://127.0.0.1:8000
```

These are different origins.

CORS will be configured when we integrate the React frontend.

---

# 4.34 Chapter Completion

Completed:

- [x]  FastAPI application
- [x]  Root endpoint
- [x]  Health endpoint
- [x]  Uvicorn server
- [x]  Swagger documentation
- [x]  Pydantic request model
- [x]  POST `/api/chat`
- [x]  Ollama service
- [x]  FastAPI → Ollama → Qwen3 pipeline
- [x]  API testing through Swagger
- [x]  Basic error understanding

Current pipeline:

```
POST /api/chat
       ↓
FastAPI
       ↓
ChatRequest
       ↓
Ollama Service
       ↓
Qwen3 4B
       ↓
Response
```

