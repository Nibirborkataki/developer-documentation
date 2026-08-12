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