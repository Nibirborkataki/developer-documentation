# Directory Structure

## 3.1 Introduction

The Factsheet download Automation project follows a modular directory structure were different responsibilities are separated into dedicated folder and files.

This structure makes the application easier to understand, maintain, troubleshoot, and extends. Since each AMC can have a different websites structure and documents retrieval process, keeping the AMC-specific logic separate prevents changes in one implementations from affecting the rest of the applications.

The overall structures can be represents as:

```
Factsheet Automation/
│
├── sites/
│   ├── aditya_birla.py
│   ├── hdfc.py
│   ├── sbi.py
│   ├── axis.py
│   └── ...
│
├── factsheets/
│   └── Downloaded documents
│
├── logs/
│   └── Execution logs
│
├── main.py
├── config.py
├── download_utils.py
├── extract_urls.py
└── requirements.txt
```

_Notes: The exact files and AMC modules may vary as automation is expanded or maintained.

## 3.2 `sites/`- AMC-specific Implementations

The `/sites` directory contain the individual automation modules for different **Assets Management Companies(AMCs)**

Each AMCs can have a different websites structure, navigation flows. API implementations, or method for providing factsheets. Therefore, instead of keeping all scraping logic inside a single Python file, the logic is separated into individuals modules.

For example:

```
sites/
├── aditya_birla.py
├── hdfc.py
├── sbi.py
├── axis.py
└── ...
```

Each modules is responsible for implementing the logic required to retrieve the required documents from its respective websites.

## Responsibilities of a Site Module

A typical site-specific module may handle specific tasks such as:

1. Open the AMCs websites.
2.  Navigation to the required factsheet or resource section.
3. Select the appropriate year, month, category and fund.
4. Identifying the latest available documents.
5. Extracting from the document URL.
6. Handling dynamic content or browser interactions.
7. Downloading the documents using the appropriate methods.
8. Handling website-specific exceptions.

For example, if an AMC changes its websites structure, the corresponding module can be modified independently.

```
AMC website changes
        │
        ▼
Update corresponding site module
        │
        ▼
Other AMC modules remain unaffected
```

This make the system easier to maintain as the number of supported websites increases.

## 3.3 `factsheets\` - Output Directory

The `factsheets/` directory is used as the **output location for downloading factsheets and related documents.**

After AMC-specific module successfully identifies and downloads a document, the resulting file is stored in these directory.

For example:

```
factsheets/
├── AMC_Factsheet_January_2026.pdf
├── AMC_Factsheet_February_2026.pdf
└── AMC_Factsheet_March_2026.pdf
```

The exact naming conventions depends on the requirements of the automation and the implementation used  for the respective AMCs.

Keeping downloaded files in a dedicated directory provide several advantages:

* Keeps generated files separated from application code.
* Make download documents easy to locate.
* Provide a consistent output location for downstream location.
* Make it easier to verify weather a download was successful.

## 3.4 `logs/` - Execution Logs

The `logs/` directory contains logs generated during the execution of the automation.

Since the application processes multiple websites sequentially, logging is important for understanding what happened during a particular execution.

A log can help determine:

- Which AMC process started.
- Whether the website was accessed successfully.
- Whether a factsheet was found.
- Which document was downloaded.
- Whether a download failed.
- What error occurred during execution.
- When a particular operation was completed.

A typical structure may look like:

```
logs/
├── automation.log
└── ...
```

Logging is particularly useful when the automation is executed without continuously monitoring the terminal.

For example, if one AMC fails while the remaining processes complete successfully, the log can be checked to identify the specific module and error responsible for the failure.

## 3.5 `main.py` - Application Entry Point

The `main.py` files acts as a *central entry point* of the automation.

Instead of executing every AMC module individually, the user can start the complete process through `main.py`.

The main execution flow can represented as:

```
main.py
   │
   ▼
run_all()
   │
   ├── AMC 1
   ├── AMC 2
   ├── AMC 3
   ├── AMC 4
   └── ...
```

The orchestrator is responsible for triggering the required site-specific download functions.

This provides a centralized execution point and avoids the needs to manually run each individuals script.

Why `main.py` is important

Without a central entry point, the user would need to execute multiple scripts seperately.

With `main.py`:

```Bash
python main.py
```

the complete automation can be initiated from a single command.

## 3.6 `config.py`- Configuration Management

The config.py file is used to maintain *configuration-related values and settings* required by the automation.

Keeping configuration values separate from the core automation logic makes the application easier to modify and maintain.

Depending on the implementation, configuration can include values such as:

* Output directory paths.
* Log directory paths.
* Website-related settings.
* File naming configurations.
* Other reusable application settings.

For example:
```Bash
config.py
     │
     ├── Output configuration
     ├── Logging configuration
     └── Application settings
```

Separating these values from the site-specific logic reduces hard-coded values throughout the project.

## 3.7 `download_utils.py` - Download Utilities

The `download_utils.py` file contains *reusable functions related to downloading files.*

Since multiple AMC modules need to download documents, implementing the same download logic repeatedly would result in unnecessary code duplication.

Instead, common functionality can be placed in `download_utils.py` and reused by different site modules.

For example:

```
sites/axis.py
      │
      └──────┐
             ▼
      download_utils.py
             │
             ▼
        Download PDF
```

Typical responsibilities may include:
* Sending download requests.
* Saving files to the required directory.
* Handling download-related errors.
* managing file paths,
* Handling common download conditions.
* Supporting reusable download operations.

This approach follows the principle of code reusability and keeps the site-specific modules focused on website interaction rather than common file operations.

## 3.8 `extract_urls.py` - URL Extraction Utilities

The `extract_urls.py` files provides functionality related to *extracting and processing documents URLSs*

Different AMC websites may expose document link in different ways. The required PDF URL may be available directly in an HTML element, generated dynamically, or obtained through another page or request.

The general flow can be represented as: 

```
AMC Website
     │
     ▼
Find document reference
     │
     ▼
Extract URL
     │
     ▼
Validate/process URL
     │
     ▼
Download document
```

Separating URL-related functionality helps keep the individual AMC modules cleaner and easier to read.

## 3.9 `requirenments.txt` - Python Dependencies

The `requirenments.txt` file contains the Python package required to run the automation.

Instead of installing dependencies individually, the required packages can be installed together using:

```Bash
pip install -r requirements.txt
```

This make the project easier to set up on a new machine or development environment.

The file also helps maintain consistency between different environments by providing a defined list of project dependencies.

## 3.10 Relationship Between Components 

The major components work together as follows:

```Bash
                    main.py
                       │
                       ▼
                    run_all()
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    sites/AMC1     sites/AMC2     sites/AMC3
        │              │              │
        └──────────────┼──────────────┘
                       ▼
              Shared Utilities
             ┌─────────┴─────────┐
             ▼                   ▼
     download_utils.py     extract_urls.py
             │                   │
             └─────────┬─────────┘
                       ▼
                 factsheets/
                       │
                       ▼
                Downloaded Files

              Execution Information
                       │
                       ▼
                    logs/
```

The separation between the **orchestrator**, **AMC-specific modules**, **shared utilities**, **configuration**, **logs**, and **output files** allows each component to have a clear responsibility.

## 3.11 Summary 

The directory structure is designed around the principle of **separating website-specific logic from common application functionality.**

The `main.py` file acts as the orchestrator, the `sites/` directory contains individual AMC implementations, shared utilities handle reusable operations, `factsheets/` store the generated output, and `logs/` provide execution information.

This organization allows the automation to grow to support additional AMC websites while keeping the existing implementation manageable and easier to maintain.



