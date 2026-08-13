# The Core Workflow

## 4.1 Introduction

The factsheet download Automation follows a structured workflow in which each component has a specific responsibility.

The overall process is controlled by `main.py`, while the actual website interaction and document retrieval logic is handled by the individual modules inside the `sites/` directory.

This separation is important because every AMC website can have a different structure and may require a different approach for locating and downloading factsheet.

The high-level workflow is:

```
                main.py
                   │
                   ▼
             Start Automation
                   │
                   ▼
          Call AMC-specific module
                   │
                   ▼
             sites/<amc>.py
                   │
                   ▼
          Access AMC Website/API
                   │
                   ▼
          Locate Required Document
                   │
                   ▼
            Extract Document URL
                   │
                   ▼
              Download File
                   │
                   ▼
             Save to factsheets/
                   │
                   ▼
             Return to main.py
                   │
                   ▼
           Continue with next AMC
```


## 4.2 Starting the Automation

The complete automation is initiated through `main.py` file.

Instead of manually executing every AMC- specific script, the user runs the main application.

``
```Bash
python main.py
```
The `main.py` file acts as the **orchestrator** of the application.

Its primary responsibility is to determine which **download functions need to be executed and in what sequence**

It does not contain the detailed scraping logic for every AMC.

This keeps the entry point simple and makes the application easier to manage.


## 4.3 Roles of `main.py`

The main purpose of `main.py` is to act a **caller and coordinator.**

For example, the application may have function such as:

```python
download_aditya_birla()
download_hdfc()
download_sbi()
download_axis()
```

The orchestrator can call these function sequentially.

Conceptually:

```
main.py
   │
   ├── download_aditya_birla()
   │
   ├── download_hdfc()
   │
   ├── download_sbi()
   │
   ├── download_axis()
   │
   └── ...
```

The important design principle here is that `main.py` does not need to know how each AMC website works.

It only needs to know:

> "Call the download function for this AMC."

The implementation details remain inside the corresponding module.

## 4.4 Separation of Concerns

The project follows the principle of separation of concerns.

This means that different parts of the application are responsible for different tasks.

For example:

|Component|Responsibility|
|---|---|
|`main.py`|Orchestrates the complete execution|
|`sites/<amc>.py`|Handles AMC-specific website logic|
|`download_utils.py`|Handles reusable download operations|
|`extract_urls.py`|Handles URL extraction/processing|
|`config.py`|Provides configuration|
|`factsheets/`|Stores downloaded documents|
|`logs/`|Stores execution information|

This separation prevents the main application from becoming a large file containing hundreds or thousands of lines of website-specific code.

## 4.5 Why the Heavy Lifting Is Done Inside `sites/`

Every AMC website can have a completely different implementation.

For example, one website might provide a direct PDF link:

```
Website
   ↓
PDF link
   ↓
Download
```

Another website might require:

```
Website
   ↓
Select Year
   ↓
Select Category
   ↓
Find Latest Factsheet
   ↓
Extract PDF URL
   ↓
Download
```

Another implementation may require browser automation:

```
Website
   ↓
Open Browser
   ↓
Navigate
   ↓
Interact with Page
   ↓
Wait for Dynamic Content
   ↓
Extract Document
   ↓
Download
```

Because of these differences, putting all the logic inside `main.py` would make the code difficult to maintain.

Instead, each AMC gets its own module:

```
sites/
├── amc_1.py
├── amc_2.py
├── amc_3.py
└── ...
```

Each module is responsible for understanding how its particular website works.


## 4.6 Lifecycle of a Single Download

The following describes the typical lifecycle of one factsheet download.

### Step 1: Function Is Called

The process begins when `main.py` calls the appropriate AMC-specific function.

```
main.py
   │
   ▼
download_<amc>()
```

The function is located inside the corresponding module in the `sites/` directory.


### Step 2: Website Is Accessed

The AMC-specific module accesses the required website.

Depending on the implementation, this can be done using:

- Direct HTTP requests
- Playwright
- Selenium
- API requests
- Other web-scraping techniques

The module determines which approach is appropriate for that particular website.


### Step 3: Required Page or Resource Is Located

Once the website is accessed, the automation navigates to the relevant section.

For example:

```
AMC Website
     │
     ▼
Resources
     │
     ▼
Factsheets
     │
     ▼
Latest Available Document
```

Some websites may require additional navigation, filters, dropdown selections, pagination, or dynamic content handling.

These operations are handled entirely by the corresponding site module.


### Step 4: Latest Document Is Identified

The automation identifies the required factsheet based on the logic implemented for that AMC.

Depending on the website, this may involve:

- Checking publication dates.
- Selecting the latest month.
- Selecting the latest year.
- Finding the latest document in a list.
- Reading document titles.
- Extracting links from HTML.
- Processing API responses.

The goal is to identify the correct document rather than simply downloading the first available file.


### Step 5: Document URL Is Extracted

After identifying the required document, the automation obtains its URL.

For example:

```
AMC Website
     │
     ▼
Factsheet Entry
     │
     ▼
Document Link
     │
     ▼
PDF URL
```

Where common URL-processing functionality is required, utilities such as `extract_urls.py` can be used.


### Step 6: Document Is Downloaded

Once the document URL has been obtained, the download process begins.

The site-specific module can use the appropriate download utility or method to retrieve the document.

Conceptually:

```
PDF URL
   │
   ▼
Download Utility
   │
   ▼
PDF File
```

The downloaded document is then saved to the designated output directory.


### Step 7: File Is Stored

The final document is stored inside:

```
factsheets/
```

For example:

```
factsheets/
└── Latest_Factsheet.pdf
```

The exact filename and organization depend on the requirements of the automation.


### Step 8: Process Completes

Once the download is successfully completed, control returns to the orchestrator.

The `main.py` process then continues with the next AMC.

```
AMC 1
  ↓
Download Complete
  ↓
AMC 2
  ↓
Download Complete
  ↓
AMC 3
  ↓
...
```

This allows the complete set of download routines to execute in a single run.


# 4.7 Error Handling in `main.py`

One of the important aspects of the workflow is the use of `try/except` blocks in `main.py`.

Because the automation interacts with external websites, failures are expected to occur occasionally.

Possible causes include:

- Website downtime.
- Network problems.
- Website structure changes.
- Missing documents.
- Invalid URLs.
- Timeout errors.
- Browser automation failures.
- Unexpected website responses.

If an error from one AMC were allowed to propagate without being handled, it could potentially terminate the entire automation.

For example:

```
AMC 1
  ✓ Success

AMC 2
  ✗ Error

AMC 3
  ⛔ Never executed
```

This is undesirable because a problem with one website should not prevent other websites from being processed.


# 4.8 Preventing Cascading Failures

To avoid this problem, the execution of individual AMC routines can be isolated using `try/except`.

Conceptually:

```
try:
    download_amc_1()
except Exception as e:
    log_error(e)

try:
    download_amc_2()
except Exception as e:
    log_error(e)

try:
    download_amc_3()
except Exception as e:
    log_error(e)
```

The important concept is that an exception from one AMC is handled before the automation moves to the next one.

The workflow therefore becomes:

```
              Start
                │
                ▼
              AMC 1
                │
          ┌─────┴─────┐
          │           │
       Success       Error
          │           │
          │       Log Error
          │           │
          └─────┬─────┘
                ▼
              AMC 2
                │
          ┌─────┴─────┐
          │           │
       Success       Error
          │           │
          │       Log Error
          │           │
          └─────┬─────┘
                ▼
              AMC 3
                │
               ...
```

This is known as **failure isolation**.

A failure in one module does not automatically become a failure for the entire automation process.


# 4.9 Why Error Isolation Is Important

The automation depends on multiple external websites that are outside the direct control of the application.

Therefore, it is not realistic to assume that every AMC will always be available and behave exactly as expected.

For example:

```
40 AMC processes
       │
       ▼
39 successful
1 failed
```

The automation should still be considered largely successful because the failure was isolated to one implementation.

Without error isolation:

```
AMC 1 ✓
AMC 2 ✓
AMC 3 ✗
      │
      ▼
Automation stops
      │
      ▼
Remaining AMCs not processed
```

With error isolation:

```
AMC 1 ✓
AMC 2 ✓
AMC 3 ✗ → Error logged
AMC 4 ✓
AMC 5 ✓
...
```

This significantly improves the reliability of the overall process.


## 4.10 Complete End-to-End Workflow

Putting all the components together, the complete lifecycle looks like this:

```
                         User
                          │
                          ▼
                     python main.py
                          │
                          ▼
                       main.py
                    Orchestrator
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
       AMC Module      AMC Module      AMC Module
          │               │               │
          ▼               ▼               ▼
       Website/API     Website/API     Website/API
          │               │               │
          ▼               ▼               ▼
     Locate Latest    Locate Latest    Locate Latest
      Factsheet        Factsheet        Factsheet
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                  Extract Document URL
                          │
                          ▼
                   Download Process
                          │
                          ▼
                     factsheets/
                          │
                          ▼
                   Downloaded Files

        If an AMC fails:
                │
                ▼
          try/except
                │
                ▼
           Log Error
                │
                ▼
        Continue Next AMC
```



## 4.11 Summary

The core workflow separates **orchestration** from **implementation**.

`main.py` acts as the central coordinator and is responsible for calling the required download functions. The actual website interaction, document identification, URL extraction, and download logic is handled by the individual modules inside `sites/`.

Shared utilities provide reusable functionality, while `try/except` handling in the orchestration layer helps isolate failures between different AMC processes.

As a result, the automation can continue processing other websites even when one implementation encounters an error. This architecture provides a strong foundation for maintaining and expanding the system as additional AMC websites and download requirements are introduced.