# Chapter 5: Shared Utilities & Helpers

## 5.1 Overview

The Factsheet Download Automation project contains several shared utility modules that provide common functionality required by multiple AMC-specific automation scripts.

Instead of implementing the same logic repeatedly inside every file under the `sites/` directory, common operations are centralized into reusable helper modules.

The main shared utility files are:

- `download_utils.py`
- `extract_urls.py`
- `config.py`

These modules follow the principle of **separation of concerns**.

The AMC-specific modules are responsible for understanding how a particular website works, while the shared utilities handle common operations such as downloading files, extracting URLs, managing configuration values, and other reusable tasks.

This approach provides several advantages:

- Reduces duplicate code.
- Makes AMC modules easier to read.
- Makes maintenance easier.
- Reduces the possibility of inconsistent implementations.
- Allows common functionality to be fixed in one location.
- Makes it easier to add new AMC integrations.

---

# 5.2 Why Shared Utilities Are Important

A typical AMC module may need to perform several common operations.

For example:

1. Find the factsheet URL.
2. Extract the URL from the webpage.
3. Download the PDF.
4. Save the file.
5. Handle errors.
6. Use a predefined output location.

If every AMC module implements these operations independently, the project can quickly become difficult to maintain.

For example, without shared utilities, multiple files might contain their own versions of:

```python
requests.get(url)
```

or their own URL extraction and file-saving logic.

This creates unnecessary duplication.

Instead, the project provides reusable helper functions that can be called from the AMC modules.

For example:

```python
from download_utils import download_file

download_file(pdf_url, output_path)
```

The AMC module only needs to determine **which file should be downloaded**.

The shared utility handles **how the file is downloaded**.

This keeps responsibilities clearly separated.

# 5.3 `download_utils.py`

## Purpose

`download_utils.py` contains reusable functions related to downloading files.

The purpose of this module is to provide a centralized implementation for downloading documents such as:

- Factsheet PDFs
- Reports
- Financial documents
- Other downloadable resources

Instead of implementing HTTP requests and file-writing logic separately in every AMC module, developers can use the functions provided by this module.

---

## Responsibility

The main responsibilities of `download_utils.py` include:

- Accepting a document URL.
- Sending the request to the URL.
- Receiving the response.
- Handling download-related errors.
- Writing the downloaded content to the required location.
- Providing consistent download behaviour across AMC modules.

The exact functions available in this file depend on the current implementation of the project.

Developers should check `download_utils.py` before creating a new download function.

---

## Example Usage

Suppose an AMC module has already identified the PDF URL:

```
pdf_url = "https://example.com/factsheet.pdf"
```

Instead of writing download logic manually, the existing utility can be used:

```
from download_utils import download_file

download_file(
    pdf_url,
    "factsheets/example_factsheet.pdf"
)
```

The AMC-specific module is therefore responsible only for finding the correct URL.

The shared utility is responsible for downloading the document.

---

## Why Developers Should Reuse It

A developer should avoid writing a new download implementation inside every AMC module.

### Avoid:

```
def download_amc_pdf(url, filename):
    response = requests.get(url)

    with open(filename, "wb") as file:
        file.write(response.content)
```

If similar code already exists in `download_utils.py`, creating another implementation introduces unnecessary duplication.

### Prefer:

```
from download_utils import download_file

download_file(url, filename)
```

This ensures that improvements or bug fixes made to the common download functionality can automatically benefit all AMC modules using it.

---

# 5.4 `extract_urls.py`

## Purpose

`extract_urls.py` provides functionality related to identifying or extracting URLs from webpage content.

AMC websites do not always expose their factsheet links in the same way.

Depending on the website, the PDF URL may be available through:

- An HTML `<a>` element.
- A button.
- A dynamically generated element.
- Page source.
- Embedded content.
- JavaScript-generated data.
- API responses.

The URL extraction utilities help standardize this process where common extraction logic can be reused.

---

## Responsibility

The responsibilities of `extract_urls.py` may include:

- Finding URLs from HTML or text.
- Identifying document links.
- Filtering relevant URLs.
- Extracting PDF or downloadable resource URLs.
- Supporting AMC modules where URL extraction follows a reusable pattern.

The AMC-specific module should still contain website-specific logic when necessary.

---

## Example Usage

If a page contains multiple links:

```
html_content = get_page_content()

urls = extract_urls(html_content)
```

The resulting URLs can then be filtered to locate the required document:

```
pdf_urls = [
    url for url in urls
    if url.lower().endswith(".pdf")
]
```

The selected URL can then be passed to the download utility:

```
from download_utils import download_file

download_file(
    pdf_urls[0],
    "factsheets/latest_factsheet.pdf"
)
```

This creates a simple pipeline:

```
Webpage
   ↓
URL Extraction
   ↓
Identify PDF URL
   ↓
Download Utility
   ↓
Saved Factsheet
```

---

# 5.5 `config.py`

## Purpose

`config.py` is used to centralize configuration values required by the automation system.

Configuration values should generally not be scattered throughout multiple AMC modules.

Centralizing these values makes the project easier to configure and maintain.

---

## Typical Configuration Responsibilities

Depending on the current implementation, `config.py` can contain values such as:

- Output directories.
- Factsheet storage paths.
- Log locations.
- Browser configuration.
- Timeout values.
- Common URLs.
- Other project-level settings.

For example:

```
FACTSHEET_DIR = "factsheets"
LOG_DIR = "logs"
```

An AMC module can then use these values:

```
from config import FACTSHEET_DIR

output_path = f"{FACTSHEET_DIR}/latest_factsheet.pdf"
```

Instead of hardcoding:

```
output_path = "factsheets/latest_factsheet.pdf"
```

throughout the project.

---

# 5.6 Example: Combining Shared Utilities

A typical AMC implementation can combine the shared utilities with website-specific logic.

For example:

```
from download_utils import download_file
from config import FACTSHEET_DIR

def download_example_amc():

    # Website-specific logic
    pdf_url = find_latest_factsheet_url()

    if not pdf_url:
        print("Factsheet URL not found")
        return

    # Common download logic
    output_path = f"{FACTSHEET_DIR}/example_amc_factsheet.pdf"

    download_file(
        pdf_url,
        output_path
    )
```

Here the responsibilities are clearly separated.

### AMC Module

Responsible for:

- Opening the AMC website.
- Navigating the required pages.
- Finding the latest factsheet.
- Identifying the correct PDF URL.

### Shared Utilities

Responsible for:

- Downloading the file.
- Handling common download operations.
- Providing reusable functionality.

### Configuration

Responsible for:

- Providing common project settings.
- Defining shared paths and configuration values.

---

# 5.7 Recommended Development Approach

When adding a new AMC module, developers should follow this order:

### Step 1 — Understand the AMC Website

First identify:

- Factsheet page.
- Latest factsheet.
- HTML structure.
- Download link.
- Whether JavaScript is required.
- Whether an API is involved.

---

### Step 2 — Implement Website-Specific Logic

Create the AMC-specific module under:

```
sites/
```

For example:

```
sites/
└── new_amc.py
```

The module should focus primarily on the logic required to interact with that particular website.

---

### Step 3 — Reuse Existing Utilities

Before creating a new helper function, check:

```
download_utils.py
extract_urls.py
config.py
```

Determine whether the required functionality already exists.

If it does, reuse it.

---

### Step 4 — Add New Shared Functionality Only When Necessary

If the same functionality is required by multiple AMC modules and does not already exist, consider adding it to the appropriate utility module.

For example:

```
download_utils.py
```

should contain download-related functionality.

Similarly:

```
extract_urls.py
```

should contain reusable URL extraction functionality.

This prevents utility files from becoming mixed collections of unrelated functions.

---

# 5.8 Separation of Responsibilities

The project can be viewed as three main layers:

```
┌───────────────────────────────┐
│        AMC Site Modules       │
│       sites/*.py              │
│                               │
│ Website-specific logic        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Shared Utilities        │
│                               │
│ download_utils.py             │
│ extract_urls.py               │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│          Configuration         │
│                               │
│ config.py                     │
└───────────────────────────────┘
```

This structure makes it possible to modify one part of the system without unnecessarily changing the others.

For example, if the download mechanism needs improvement, the developer can update `download_utils.py` rather than modifying every AMC module.

---

# 5.9 Benefits of the Shared Utility Architecture

Using shared utilities provides several long-term benefits.

### Maintainability

Common functionality exists in one location, making maintenance easier.

### Reusability

The same function can be used by multiple AMC modules.

### Consistency

All modules follow the same approach for common operations.

### Reduced Code Duplication

Developers do not need to repeatedly write the same download or URL-processing logic.

### Easier Debugging

When a common operation fails, developers can investigate the shared utility instead of checking dozens of AMC modules individually.

### Scalability

As the number of supported AMC websites grows, the project can continue adding website-specific modules without duplicating the entire download infrastructure.

---

# 5.10 Developer Guidelines

When working on the automation project, follow these guidelines:

1. **Check existing utilities before writing new helper functions.**
2. **Keep AMC-specific logic inside the corresponding `sites/` module.**
3. **Use `download_utils.py` for common download operations.**
4. **Use `extract_urls.py` for reusable URL extraction functionality.**
5. **Use `config.py` for shared configuration values.**
6. **Avoid hardcoding project-wide paths when a configuration value already exists.**
7. **Avoid copying the same function into multiple AMC modules.**
8. **If a new helper is useful for multiple modules, consider adding it to the appropriate shared utility.**
9. **Keep utility functions focused on a single responsibility.**
10. **Test changes to shared utilities carefully because multiple AMC modules may depend on them.**

---

## 5.11 Summary

The shared utility layer is an important part of the Factsheet Download Automation architecture.

The `sites/` modules determine **how to find the factsheet on a specific AMC website**, while the shared utilities provide reusable functionality for **processing URLs, downloading documents, and managing configuration**.

This separation keeps the automation system modular and allows new AMC integrations to be developed without repeatedly implementing the same underlying functionality.

The preferred development pattern is:

```
Identify Factsheet
        ↓
Website-Specific Logic
        ↓
Extract / Validate URL
        ↓
Shared Download Utility
        ↓
Configured Output Location
        ↓
Downloaded Factsheet
```
