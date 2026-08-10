# Getting Started (Installation & Setup)

This chapter provides a step-by-step guide to setting up the **Factsheet Automation** project on your local machine. By following these instructions, we will configure the required Python environment, install necessary dependencies, and prepare the browser automation tools needed to fetch factsheets.

## 2.1 Prerequisites

Before you begin, ensure that your system meets the following requirements:

* **Python 3.8 or higher**: The core programming language used for this script. You can verify if Python is installed by opening your terminal or command prompt and running:

```bash

python --version

```

* **Git (Optional but recommended)**: For cloning the repository and managing version control.

## 2.2 Setting Up the Environment

To avoid conflicts with other Python projects on your computer, it is highly recommended to run this automation within a **Virtual Environment**. Think of a virtual environment as a self-contained sandbox for this specific project.

### Step 1: Navigate to the Project Directory

Open your terminal (or Command Prompt / PowerShell on Windows) and navigate to the folder where the project is located:

```bash

cd path/to/Fund_Manager_automation

```

### Step 2: Create a Virtual Environment

Run the following command to create a new virtual environment named `.venv`:

```
bash

python -m venv .venv
```

_Note: This command creates a new folder named `.venv` containing a standalone Python installation._

### Step 3: Activate the Virtual Environment

You must activate the virtual environment every time you want to run or work on the project. The command varies depending on your operating system:

- **On Windows (Command Prompt):**
    
    ```
    ```cmd
    
    .venv\Scripts\activate.bat
    ```

- **On Windows (PowerShell):**

    ```powershell
    
    .venv\Scripts\Activate.ps1
    ```

- **On macOS and Linux:**
    
    ```
    bash
    
    source .venv/bin/activate
    ```

_How to verify:_ Once activated, your terminal prompt should change to show `(.venv)` at the beginning of the line.

## 2.3 Installing Dependencies

With the virtual environment activated, you need to install the project's third-party libraries. This project uses packages like `requests` (for making network calls) and `playwright` (for navigating complex, JavaScript-heavy AMC websites).

Run the following command to install all required libraries from the `requirements.txt` file:

```bash

pip install -r requirements.txt
```
## 2.4 Setting Up Playwright Browsers

Because this project uses **Playwright** to interact with websites exactly as a human would, Playwright needs to download its own specialized browser binaries (Chromium, Firefox, WebKit) to function correctly.

Run the following command in your terminal (make sure your `.venv` is still active):

```
bash

playwright install
```


_Note: This step might take a few minutes as it downloads several hundred megabytes of browser files. You only need to do this once._

## 2.5 Verifying the Installation

To ensure everything is set up correctly, you can perform a dry run.

1. Ensure your terminal shows the `(.venv)` prefix.
2. Execute the main script:
    ```
    
    bash
    
    python main.py
    ```

1. You should see an output in the console starting with `Starting automation...` and eventually concluding with `All tasks completed!`. Check the `factsheets/` directory to verify that files are being downloaded.