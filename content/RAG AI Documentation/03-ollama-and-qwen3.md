
# Chapter 3 — Ollama + Qwen3

## 3.1 Objective

The objective of this chapter is to establish communication between our Python application and the locally running Qwen3 language model through Ollama.

The pipeline we want is:

```text
Python
  ↓
Ollama
  ↓
Qwen3 4B
  ↓
Generated response
```

# 3.2 What is an LLM?

LLM stands for:

> Large Language Model

An LLM is a machine learning model trained to understand and generate human language.

In this project, Qwen3 is responsible for generating the final natural-language response.

---

# 3.3 What is Inference?

Inference is the process of using a trained model to generate an output from an input.

For example:

```
Input:
"What is RAG?"

        ↓
     Qwen3

        ↓

Output:
"RAG stands for Retrieval-Augmented Generation..."
```

The model is not being trained during this process.

It is performing inference using an already-trained model.

---

# 3.4 Ollama's Role

Ollama acts as the local model runtime.

Our application does not directly manage the Qwen model's low-level execution.

Instead:

```
Python
   ↓
Ollama
   ↓
Qwen3
```

Ollama handles communication with the local model.

---

# 3.5 Python Ollama Library

The Python application communicates with Ollama using the `ollama` package.

Import:

```
import ollama
```

---

# 3.6 Test File

Create:

```
backend/
└── test_ollama.py
```

Code:

```
import ollama


response = ollama.chat(
    model="qwen3:4b",
    messages=[
        {
            "role": "user",
            "content": "Explain what Retrieval-Augmented Generation is in simple terms."
        }
    ]
)

print(response["message"]["content"])
```

---

# 3.7 Understanding the Code

## Import Ollama

```
import ollama
```

This imports the Python client used to communicate with the Ollama runtime.

---

## Calling the Model

```
ollama.chat(...)
```

This sends a chat request to Ollama.

---

## Selecting the Model

```
model="qwen3:4b"
```

This tells Ollama which installed model should process the request.

The model name must match the model installed on the machine.

For example:

```
qwen3:4b
```

---

## Messages

```
messages=[
    {
        "role": "user",
        "content": "..."
    }
]
```

The `messages` parameter represents the conversation input.

The role here is:

```
user
```

The content contains the question.

---

# 3.8 Response Object

The response returned by Ollama contains information about the generated message.

We access the generated content using:

```
response["message"]["content"]
```

Then:

```
print(response["message"]["content"])
```

prints the model's answer.

---

# 3.9 Important Naming Difference

A previous error occurred because the wrong model parameter/name was used.

Incorrect:

```
model="qwen:4b"
```

Correct:

```
model="qwen3:4b"
```

The model name must exactly match the installed model.

Verify using:

```
ollama list
```

---

# 3.10 First Successful Test

The Python application successfully communicated with Qwen3 through Ollama.

The flow was:

```
test_ollama.py
      ↓
Python Ollama Client
      ↓
Ollama
      ↓
Qwen3 4B
      ↓
Response
      ↓
Python
```

This proves that the local AI layer is working.

---

# 3.11 Why Does the First Request Take Time?

The first inference can take longer because the model may need to be loaded into system memory or GPU memory.

The process is approximately:

```
Request
 ↓
Load model
 ↓
Process prompt
 ↓
Generate tokens
 ↓
Return response
```

Later requests may be faster if the model remains loaded.

---

# 3.12 What Are Tokens?

LLMs process text as tokens.

A token may represent:

- A word
- Part of a word
- Punctuation
- Characters or groups of characters

The model processes tokens rather than directly processing complete sentences as humans perceive them.

---

# 3.13 Generation

Qwen3 generates the answer token-by-token.

Conceptually:

```
Prompt
 ↓
Token 1
 ↓
Token 2
 ↓
Token 3
 ↓
...
 ↓
Complete response
```

---

# 3.14 Current AI Pipeline

At the end of Chapter 3:

```
Python Application
        │
        ▼
      Ollama
        │
        ▼
    Qwen3 4B
        │
        ▼
   AI Response
```

However, this is **not RAG yet**.

Currently, Qwen3 answers using the model's existing knowledge.

---

# 3.15 Why This Is Not RAG

Our current system:

```
Question
   ↓
Qwen3
   ↓
Answer
```

A RAG system will eventually be:

```
Question
   ↓
Embedding
   ↓
Vector Search
   ↓
Relevant Document Chunks
   ↓
Context
   ↓
Qwen3
   ↓
Grounded Answer
```

The retrieval step has not been implemented yet.

---

# 3.16 Interview Concepts

### What is inference?

Running a trained model to generate an output from an input.

### What is token generation?

The process by which the language model generates a response incrementally as tokens.

### What is the role of Ollama?

Ollama manages local model execution and provides an interface for applications to interact with local models.

### Why Qwen3?

Qwen3 provides a capable local language model while keeping the project API-free and locally executable.

### Is Qwen3 the embedding model?

No.

Qwen3 is used for generation.

The embedding model is:

```
nomic-embed-text
```

---

# 3.17 Chapter Completion

Completed:

- [x]  Ollama installed
- [x]  Qwen3 downloaded
- [x]  Python Ollama package installed
- [x]  Python → Ollama communication tested
- [x]  Qwen3 response successfully generated

Next:

> Integrate the local model with FastAPI.