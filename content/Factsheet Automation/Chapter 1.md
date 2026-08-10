# Overview

## 1.1 Introduction

The **Factsheet Download Automation** project is a Python-based automation suite designed to streamline the retrieval of mutual fund data, specifically factsheets and related financial documents, from various Asset Management Company (AMC) websites in India.

Manually navigating multiple AMC portals to locate and download monthly or daily reports is a time-consuming and error-prone process. Each AMC may use a different website structure, URL pattern, API, or document retrieval mechanism.

This project centralizes and automates the document retrieval process, allowing factsheets to be downloaded systematically through a single execution point.

The automation is designed to handle different website architectures while maintaining a consistent downloading and file management process.

---

## 1.2 Core Objective

The primary objective of the system is to provide a **robust, scalable, and modular framework** for scraping, fetching, and downloading financial documents from a large number of mutual fund websites.

The system is designed to:

- Automate the retrieval of the latest available factsheets.
- Reduce manual effort involved in collecting documents.
- Handle different website structures and document retrieval methods.
- Extract and identify the required document URLs.
- Download documents automatically to the designated location.
- Maintain a consistent file storage process.
- Handle website-specific errors without interrupting the complete automation process.
- Provide a modular architecture that makes individual AMC implementations easier to maintain.

---

## 1.3 Technologies Used

The automation primarily uses **Python** along with web automation and web scraping technologies.

Key technologies and tools include:

- **Python** – Primary programming language used to develop the automation.
- **Playwright** – Used for browser automation and handling dynamic websites.
- **Selenium** – Used for browser-based automation where required.
- **Web Scraping** – Used to extract document information and URLs from web pages.
- **HTML/CSS Selectors** – Used to locate and interact with specific elements on websites.
- **File Handling** – Used to download, rename, organize, and manage retrieved documents.
- **PDF Processing** – Used where additional processing or validation of downloaded documents is required.

Different AMC websites may require different approaches depending on their underlying architecture. Some implementations rely on direct URL extraction, while others require browser automation, DOM parsing, or API/network request handling.

---

## 1.4 High-Level Architecture

The project is built on a highly modular architecture to ensure easy maintenance and scalability:

### Orchestrator

The `main.py` file acts as the central entry point of the application.

It contains the `run_all()` function, which sequentially triggers the download routines implemented for the configured mutual fund websites.

This provides a single execution point for running the complete automation process.

### Site-Specific Modules

To keep the code clean and maintainable, the scraping and downloading logic for each individual AMC is isolated into its own dedicated Python file

`(e.g., `sites/hdfc.py`, `sites/sbi.py`).`

This means if an AMC changes its website layout, only that specific module needs to be updated.

### Output Directory (`factsheets/`)**: 
The designated location where all successfully fetched documents are saved.

* **Shared Utilities**: 
  Files like `download_utils.py` and `extract_urls.py` provide common helper functions (like downloading a file from a URL, handling retries, or parsing text) that can be reused across different AMC modules.


## 1.5 Typical Workflow

When the script is executed:

1. The user runs `main.py`.

2. The orchestrator iterates through the list of imported AMC download functions (e.g., `download_aditya_birla()`, `download_hdfc()`).

3. Each function executes its custom logic to navigate the target AMC's website or API.

4. The target documents are identified, downloaded, and saved to the local file system.

5. Errors are caught individually per AMC, ensuring that a failure in one fund house's script does not halt the entire automation process.