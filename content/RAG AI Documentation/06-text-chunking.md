# Chapter 6 — Text Chunking

## 6.1 Overview

In the previous chapter, we extracted text from PDF documents using PyMuPDF.

The extraction pipeline was:

```text
PDF
 ↓
PyMuPDF
 ↓
Page-level text
```

However, sending an entire PDF or an entire page directly to an embedding model is not ideal.

Instead, we divide the extracted text into smaller pieces called **chunks**.

The pipeline now becomes:

```text
PDF
 ↓
PyMuPDF
 ↓
Page Text
 ↓
Text Cleaning
 ↓
Sentence Splitting
 ↓
Chunking
 ↓
Chunks + Metadata
```

These chunks will later be converted into embeddings and stored in ChromaDB.

---

# 6.2 Why Do We Need Chunking?

A mutual fund factsheet can contain many pages of information.

For example:

```text
Page 1
Page 2
Page 3
...
Page 22
```

If we treat the entire PDF as one piece of text, several problems occur:

- The text can be very large.
    
- Retrieval becomes less precise.
    
- An embedding represents too much unrelated information.
    
- The AI may receive unnecessary context.
    
- Finding the exact information becomes harder.
    

Instead, we divide the document:

```text
Large Document
      ↓
Small Chunks
      ↓
Embeddings
      ↓
Vector Search
```

This allows the RAG system to retrieve only the relevant pieces.

---

# 6.3 What Is a Chunk?

A chunk is a smaller section of document text.

For example:

```text
Original document:

The scheme primarily invests in equity and equity-related
securities. The fund follows a growth-oriented investment
strategy and seeks long-term capital appreciation...
```

Could become:

```text
Chunk 1:
The scheme primarily invests in equity and equity-related
securities.

Chunk 2:
The fund follows a growth-oriented investment strategy and
seeks long-term capital appreciation.
```

The actual chunk size depends on the configuration.

---

# 6.4 Chunk Size

Our initial configuration is:

```python
chunk_size=1000
```

This means we target approximately 1000 characters per chunk.

Example:

```text
Chunk
├── ~1000 characters
└── Metadata
```

Chunk size is an important RAG parameter.

### If chunks are too large

```text
Large Chunk
 ↓
Too much unrelated information
 ↓
Less precise retrieval
```

### If chunks are too small

```text
Small Chunk
 ↓
Context may be lost
 ↓
Poor answers
```

Therefore, chunk size is a trade-off between:

```text
Context
   ↕
Retrieval Precision
```

For this project, `1000` characters is a reasonable starting point.

It can be tuned later based on retrieval results.

---

# 6.5 Chunk Overlap

Our configuration also uses:

```python
overlap=200
```

Overlap means that a small amount of text from the previous chunk is repeated in the next chunk.

For example:

```text
Chunk 1
────────────────────────
Sentence A
Sentence B
Sentence C
Sentence D
────────────────────────

Chunk 2
────────────────────────
Sentence D
Sentence E
Sentence F
Sentence G
────────────────────────
```

Here, `Sentence D` provides overlap.

---

# 6.6 Why Do We Need Overlap?

Imagine an important sentence is located exactly at the boundary between two chunks.

Without overlap:

```text
Chunk 1
───────
Beginning of information

Chunk 2
───────
End of information
```

The meaning can become fragmented.

With overlap:

```text
Chunk 1
───────
Beginning
Important information

Chunk 2
───────
Important information
Continuation
```

The second chunk retains some context from the previous chunk.

Therefore:

> Chunk overlap helps preserve contextual continuity between chunks.

---

# 6.7 Metadata

Every chunk should preserve information about where it came from.

Our chunk structure is:

```python
{
    "file_name": "test_factsheet.pdf",
    "page_number": 5,
    "chunk_index": 2,
    "text": "..."
}
```

### `file_name`

Identifies the source PDF.

### `page_number`

Identifies the original PDF page.

### `chunk_index`

Identifies the position of the chunk within the page.

### `text`

Contains the actual chunk content.

---

# 6.8 Why Metadata Is Important

Metadata will allow the final RAG system to provide sources.

For example:

```text
Answer:
The expense ratio is 0.75%.

Source:
test_factsheet.pdf
Page: 7
```

Without metadata, we would know the answer but not where the answer came from.

This is particularly important for financial documents because source traceability is valuable.

---

# 6.9 PDF Text Cleaning

During testing, we discovered a problem with PDF text extraction.

Some words were extracted like this:

```text
opin-
ions
```

or:

```text
informa-
tion
```

This happens because PDF files often contain line-break-based text layouts.

The PDF visually displays:

```text
opinions
```

but internally the PDF may represent it as:

```text
opin-
ions
```

---

# 6.10 The Problem We Initially Encountered

Our initial chunking pipeline was directly processing the extracted text.

As a result, we saw output such as:

```text
The information/ views / opin-
ions provided is for informative purpose...
```

and:

```text
the informa-
tion provided above...
```

This was undesirable because the chunk text was not clean.

It could potentially affect:

- sentence splitting
    
- embeddings
    
- retrieval quality
    
- final AI responses
    

---

# 6.11 Solution — Text Cleaning

We introduced a `clean_text()` function.

```python
import re


def clean_text(text: str) -> str:

    # Join words split across lines.
    text = re.sub(r"-\s*\n\s*", "", text)

    # Replace remaining line breaks with spaces.
    text = re.sub(r"\s*\n\s*", " ", text)

    # Remove excessive whitespace.
    text = re.sub(r"\s{2,}", " ", text)

    return text.strip()
```

---

# 6.12 Understanding the Regex

The first expression:

```python
r"-\s*\n\s*"
```

handles words split by a hyphen and newline.

For example:

```text
informa-
tion
```

becomes:

```text
information
```

The second expression:

```python
r"\s*\n\s*"
```

handles normal line breaks.

For example:

```text
The scheme invests
in equity securities.
```

becomes:

```text
The scheme invests in equity securities.
```

The third expression:

```python
r"\s{2,}"
```

removes excessive spaces.

---

# 6.13 Another Issue — Incorrect Regex

During development, the regex was accidentally written in an incorrect form similar to:

```python
r"-\s\*\n\s\*"
```

and:

```python
r"\s\*\n\s\*"
```

These were incorrect because the `*` characters were being escaped.

The correct expressions are:

```python
r"-\s*\n\s*"
```

and:

```python
r"\s*\n\s*"
```

This was corrected before continuing with the chunking pipeline.

---

# 6.14 Sentence Splitting

After cleaning the text, we split it into sentences.

```python
def split_into_sentences(text: str) -> list[str]:

    sentences = re.split(
        r"(?<=[.!?])\s+",
        text.strip()
    )

    return [
        sentence.strip()
        for sentence in sentences
        if sentence.strip()
    ]
```

The regular expression looks for whitespace after:

```text
.
!
?
```

For example:

```text
The fund invests in equity. It follows a growth strategy.
```

becomes:

```text
[
    "The fund invests in equity.",
    "It follows a growth strategy."
]
```

---

# 6.15 Why Sentence-Aware Chunking?

A naive chunking method could simply cut every 1000 characters:

```text
Character 1 → 1000
Character 1001 → 2000
...
```

That can cut sentences in the middle.

Instead, our implementation attempts to build chunks using sentences.

Conceptually:

```text
PDF text
 ↓
Clean
 ↓
Sentences
 ↓
Build chunks
 ↓
~1000 characters
```

This generally produces more meaningful chunks.

---

# 6.16 Final Chunking Service

Our current implementation is:

```python
import re


def clean_text(text: str) -> str:
    """
    Clean text extracted from a PDF.

    Handles:
    - Words split across PDF line breaks
    - Unnecessary line breaks
    - Excessive whitespace
    """

    text = re.sub(r"-\s*\n\s*", "", text)

    text = re.sub(r"\s*\n\s*", " ", text)

    text = re.sub(r"\s{2,}", " ", text)

    return text.strip()


def split_into_sentences(text: str) -> list[str]:
    """
    Split cleaned text into sentences.
    """

    sentences = re.split(
        r"(?<=[.!?])\s+",
        text.strip()
    )

    return [
        sentence.strip()
        for sentence in sentences
        if sentence.strip()
    ]


def chunk_pages(
    pages: list[dict],
    chunk_size: int = 1000,
    overlap: int = 200
) -> list[dict]:

    if overlap >= chunk_size:
        raise ValueError(
            "Overlap must be smaller than chunk size."
        )

    chunks = []

    for page in pages:

        text = clean_text(page["text"])

        if not text:
            continue

        sentences = split_into_sentences(text)

        current_chunk = []
        current_length = 0
        chunk_index = 0

        for sentence in sentences:

            sentence_length = len(sentence)

            if (
                current_length + sentence_length > chunk_size
                and current_chunk
            ):

                chunk_text = " ".join(
                    current_chunk
                ).strip()

                chunks.append({
                    "file_name": page["file_name"],
                    "page_number": page["page_number"],
                    "chunk_index": chunk_index,
                    "text": chunk_text
                })

                chunk_index += 1

                overlap_text = []
                overlap_length = 0

                for previous_sentence in reversed(
                    current_chunk
                ):

                    if (
                        overlap_length
                        + len(previous_sentence)
                        > overlap
                    ):
                        break

                    overlap_text.insert(
                        0,
                        previous_sentence
                    )

                    overlap_length += len(
                        previous_sentence
                    )

                current_chunk = overlap_text

                current_length = sum(
                    len(sentence)
                    for sentence in current_chunk
                )

            current_chunk.append(sentence)

            current_length += sentence_length

        if current_chunk:

            chunk_text = " ".join(
                current_chunk
            ).strip()

            chunks.append({
                "file_name": page["file_name"],
                "page_number": page["page_number"],
                "chunk_index": chunk_index,
                "text": chunk_text
            })

    return chunks
```

---

# 6.17 Testing

We created:

```text
tests/test_chunking.py
```

```python
from app.services.pdf_service import extract_text_from_pdf
from app.services.chunking_service import chunk_pages


pdf_path = "data/documents/test_factsheet.pdf"


pages = extract_text_from_pdf(pdf_path)


chunks = chunk_pages(
    pages,
    chunk_size=1000,
    overlap=200
)


print(f"Total chunks: {len(chunks)}")


for chunk in chunks:

    print(f"\n{'=' * 60}")

    print(f"FILE: {chunk['file_name']}")
    print(f"PAGE: {chunk['page_number']}")
    print(f"CHUNK: {chunk['chunk_index']}")

    print(f"{'=' * 60}")

    print(chunk["text"])
```

Run:

```powershell
python tests/test_chunking.py
```

---

# 6.18 Final Testing Result

The output successfully showed:

```text
FILE: test_factsheet.pdf
PAGE: 22
CHUNK: 0
```

followed by:

```text
FILE: test_factsheet.pdf
PAGE: 22
CHUNK: 1
```

and:

```text
FILE: test_factsheet.pdf
PAGE: 22
CHUNK: 2
```

The repeated text between chunks confirmed that the configured overlap was working.

For example:

```text
Chunk 0:
...developed through analysis of 360 ONE Mutual Fund.

Chunk 1:
The above commentary has been prepared...
```

and later:

```text
Chunk 1:
...Actual market movements may vary from the anticipated trends.

Chunk 2:
Actual market movements may vary from the anticipated trends...
```

The repeated text is intentional.

---

# 6.19 PDF-Specific Limitation

One important limitation was discovered during testing.

Mutual fund factsheets contain:

- tables
    
- headers
    
- footers
    
- disclaimers
    
- columns
    
- graphical elements
    

PyMuPDF extracts text, but it does not automatically understand the semantic structure of every table.

For example, a visual table may be extracted as a sequence of text lines.

Therefore, the current implementation focuses on:

> **Text-based RAG first.**

Advanced table reconstruction and OCR can be added later if required.

---

# 6.20 Why We Didn't Solve Everything Immediately

The goal of this project is to build a working RAG system incrementally.

Instead of trying to solve:

```text
PDF extraction
+
OCR
+
Table reconstruction
+
Chunking
+
Embeddings
+
Vector search
+
LLM
```

all at once, we separated the pipeline into stages.

Current status:

```text
PDF Extraction      ✅
Text Cleaning       ✅
Chunking            ✅
Metadata            ✅
Embeddings          Next
Vector Database     Next
```

This makes the system easier to debug and understand.

---

# 6.21 Chapter 6 Summary

The final document-processing pipeline is:

```text
PDF
 ↓
PyMuPDF
 ↓
Page Text
 ↓
clean_text()
 ↓
Sentence Splitting
 ↓
Chunking
 ↓
Overlap
 ↓
Metadata
 ↓
Chunks
```

The output of Chapter 6 becomes the input for Chapter 7.

---

# 6.22 Key Interview Concepts

### What is chunking?

Breaking large documents into smaller pieces so they can be efficiently embedded and retrieved.

### Why use overlap?

To preserve context between neighboring chunks.

### Why preserve metadata?

To identify the source document and page when returning retrieved information.

### Why clean PDF text?

PDF extraction can introduce artificial line breaks, hyphenation, and excessive whitespace.

### What happens if chunks are too large?

Retrieval becomes less precise and irrelevant information may be included.

### What happens if chunks are too small?

Important context may be lost.

### Is chunking the same as embedding?

No.

```text
Chunking:
Text → Smaller Text

Embedding:
Text → Vector
```

---

# 6.23 Chapter 6 Status

**Status: ✅ Complete**

The system is now ready to convert chunks into embeddings.