
# Chapter 5 — PDF Processing

## 5.1 Overview

The goal of this chapter is to extract readable text from PDF documents so that the text can later be processed by the RAG pipeline.

The project uses **PyMuPDF** for PDF processing.

The basic pipeline is:

```text
PDF
 ↓
PyMuPDF
 ↓
Page-by-page text extraction
 ↓
Text + Metadata
```

This chapter creates the foundation for the next stage, **Text Chunking**.

---

# 5.2 Why PDF Processing Is Required

Our application is designed to answer questions about documents such as mutual fund factsheets.

A PDF is not directly useful to the embedding model.

For example:

```text
Mutual Fund Factsheet.pdf
```

must first be converted into text:

```text
Fund Name: 360 ONE Liquid Fund

Fund Manager: Mr. Pranav Mise

Expense Ratio: ...

Portfolio:
...
```

The extracted text can then be:

```text
Cleaned
 ↓
Chunked
 ↓
Embedded
 ↓
Stored in ChromaDB
```

Therefore:

> PDF processing is the first stage of the document ingestion pipeline.

---

# 5.3 Library Used — PyMuPDF

The project uses the `PyMuPDF` Python package.

The package is imported as:

```python
import fitz
```

The library allows us to:

- open PDF files
    
- access individual pages
    
- extract text
    
- inspect page information
    
- work with PDF documents programmatically
    

---

# 5.4 Installing PyMuPDF

Inside the project's virtual environment:

```powershell
pip install pymupdf
```

The package is then available through:

```python
import fitz
```

---

# 5.5 Project Directory

The PDF processing structure is:

```text
backend/
│
├── app/
│   └── services/
│       └── pdf_service.py
│
├── data/
│   └── documents/
│       └── test_factsheet.pdf
│
└── tests/
    └── test_pdf.py
```

The separation is intentional.

### `data/documents/`

Stores documents used by the application.

### `pdf_service.py`

Contains reusable PDF extraction logic.

### `test_pdf.py`

Used to test the extraction functionality independently.

---

# 5.6 PDF Service

The PDF extraction logic is stored in:

```text
app/services/pdf_service.py
```

Current implementation:

```python
import fitz
from pathlib import Path


def extract_text_from_pdf(pdf_path: str) -> list[dict]:

    document = fitz.open(pdf_path)

    file_name = Path(pdf_path).name

    pages = []

    for page_number, page in enumerate(
        document,
        start=1
    ):

        text = page.get_text()

        pages.append({
            "file_name": file_name,
            "page_number": page_number,
            "text": text
        })

    document.close()

    return pages
```

---

# 5.7 Understanding the Code

## Import PyMuPDF

```python
import fitz
```

`fitz` is the Python interface used to interact with PyMuPDF.

---

## Import Path

```python
from pathlib import Path
```

`Path` helps us work with file paths.

We use it to extract the filename:

```python
file_name = Path(pdf_path).name
```

For:

```text
data/documents/test_factsheet.pdf
```

the result is:

```text
test_factsheet.pdf
```

---

# 5.8 Opening the PDF

```python
document = fitz.open(pdf_path)
```

This opens the PDF document.

Conceptually:

```text
PDF File
   ↓
fitz.open()
   ↓
PDF Document Object
```

The document object allows us to access its pages.

---

# 5.9 Processing Pages

We process the document page by page:

```python
for page_number, page in enumerate(
    document,
    start=1
):
```

`start=1` is important because users normally refer to PDF pages starting from page 1.

Without it, Python indexing would start from:

```text
0
```

Our application instead uses:

```text
Page 1
Page 2
Page 3
...
```

---

# 5.10 Extracting Text

The actual text extraction is performed using:

```python
text = page.get_text()
```

This extracts text from the current PDF page.

For example:

```text
PDF Page 5
       ↓
page.get_text()
       ↓
"The scheme primarily invests..."
```

---

# 5.11 Preserving Metadata

We don't store only the text.

Each page is represented as:

```python
{
    "file_name": "test_factsheet.pdf",
    "page_number": 5,
    "text": "..."
}
```

This metadata is extremely important for the later RAG pipeline.

When a chunk is retrieved, we need to know where it came from.

For example:

```text
Answer:
The fund invests primarily in equity.

Source:
test_factsheet.pdf
Page 5
```

---

# 5.12 Why Store Page Metadata?

Suppose ChromaDB retrieves:

```text
The scheme primarily invests in equity...
```

Without metadata:

```text
Where did this information come from?
```

We wouldn't know.

With metadata:

```python
{
    "file_name": "test_factsheet.pdf",
    "page_number": 5
}
```

we can identify the source.

This will later support source citations in the frontend.

---

# 5.13 Closing the Document

After extraction:

```python
document.close()
```

This releases the PDF resource.

The general lifecycle is:

```text
Open PDF
 ↓
Read pages
 ↓
Extract text
 ↓
Close PDF
```

---

# 5.14 Returning the Result

The function returns:

```python
return pages
```

where `pages` is a list.

Example:

```python
[
    {
        "file_name": "test_factsheet.pdf",
        "page_number": 1,
        "text": "..."
    },
    {
        "file_name": "test_factsheet.pdf",
        "page_number": 2,
        "text": "..."
    }
]
```

Therefore:

```text
One PDF
 ↓
Many page dictionaries
```

---

# 5.15 Testing PDF Extraction

We created:

```text
tests/test_pdf.py
```

```python
from app.services.pdf_service import extract_text_from_pdf


pdf_path = "data/documents/test_factsheet.pdf"


pages = extract_text_from_pdf(pdf_path)


print(f"Extracted pages: {len(pages)}")


for page in pages:

    print(f"\n{'=' * 60}")
    print(f"FILE: {page['file_name']}")
    print(f"PAGE: {page['page_number']}")
    print(f"{'=' * 60}")

    print(page["text"])
```

Run:

```powershell
python tests/test_pdf.py
```

---

# 5.16 Testing Result

The extraction successfully processed the mutual fund factsheet.

The output contained:

```text
FILE: test_factsheet.pdf
PAGE: 1
...
PAGE: 2
...
PAGE: 3
...
```

and the corresponding text from each page.

This confirmed that PyMuPDF was successfully extracting the document's text.

---

# 5.17 Important Limitation — Images

`page.get_text()` extracts text that exists as text inside the PDF.

It does not automatically understand every image.

For example, if a PDF page contains:

```text
[Image containing text]
```

PyMuPDF may not return the text inside that image.

Therefore:

```text
Text-based PDF
     ↓
PyMuPDF
     ↓
Works well
```

but:

```text
Scanned PDF
     ↓
Image
     ↓
PyMuPDF text extraction
     ↓
May return little/no text
```

---

# 5.18 OCR as a Future Enhancement

If scanned documents become important, OCR can be introduced later.

Possible future pipeline:

```text
PDF
 ↓
Detect image/scanned page
 ↓
OCR
 ↓
Extracted text
 ↓
Chunking
```

However, OCR is not required for the initial MVP.

The project currently focuses on text-based mutual fund factsheets.

---

# 5.19 Another Limitation — Tables

Mutual fund factsheets contain many tables.

For example:

```text
Company                Sector              Weight
--------------------------------------------------
Company A              Banking             5.23%
Company B              IT                  4.71%
Company C              Finance             3.84%
```

PyMuPDF can extract the text, but it does not necessarily reconstruct the visual table structure perfectly.

The extracted result may instead look like a sequence of text values.

This is an important limitation to remember when improving the project later.

---

# 5.20 Why We Didn't Solve Tables Immediately

The goal was to first build a working text-based RAG pipeline.

The development strategy was:

```text
PDF Extraction
      ↓
Chunking
      ↓
Embeddings
      ↓
Vector Search
      ↓
RAG
```

Once this works, table handling can be improved separately.

This prevents the project from becoming unnecessarily complicated during the initial implementation.

---

# 5.21 Important Design Decision

The PDF service accepts the PDF path as an argument:

```python
extract_text_from_pdf(pdf_path)
```

We did **not** hardcode:

```text
test_factsheet.pdf
```

inside `pdf_service.py`.

This is important because eventually PDFs will come from a frontend upload.

The future flow will be:

```text
React
 ↓
Upload PDF
 ↓
FastAPI
 ↓
Save uploaded file
 ↓
extract_text_from_pdf(uploaded_file_path)
```

Therefore, the service remains reusable.

---

# 5.22 Chapter 5 Summary

The PDF processing pipeline is:

```text
PDF
 ↓
PyMuPDF
 ↓
Open Document
 ↓
Read Each Page
 ↓
Extract Text
 ↓
Attach Metadata
 ↓
Return Page Data
```

The output becomes the input for Chapter 6.

---

# 5.23 Chapter 5 Status

**Status: ✅ Complete**

The project can successfully extract page-level text and metadata from PDF documents.

Next:

```text
Page Text
 ↓
Text Cleaning
 ↓
Chunking
```

This is covered in **Chapter 6 — Text Chunking**.