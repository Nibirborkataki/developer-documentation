
# Guide — Adding a New Fund Manager

## 6.1 Overview

One of the key design goals of the **Factsheet Download Automation** project is to make it easy to add support for a new Asset Management Company (AMC) without modifying the existing automation logic unnecessarily.

Each AMC website can have a different structure, navigation flow, API, download mechanism, or document naming convention. Therefore, the project keeps AMC-specific logic isolated inside the `sites/` directory.

When a new fund manager needs to be supported, the developer should create a new site module and connect it to the central `main.py` orchestrator.

The overall process is:

```
New AMC
   ↓
Create AMC Module
   ↓
Implement Website-Specific Logic
   ↓
Find Factsheet
   ↓
Download PDF
   ↓
Test Module
   ↓
Import into main.py
   ↓
Add to run_all()
   ↓
Run Complete Automation
```

This approach allows the project to grow without making `main.py` unnecessarily complicated.

---

# 6.2 Before Starting

Before creating a new AMC module, first inspect the target website manually.

Identify:

- The AMC's factsheet/resource page.
- Where the latest factsheet is displayed.
- How the document URL is generated.
- Whether the page requires JavaScript.
- Whether an API is involved.
- Whether pagination is required.
- Whether the PDF URL is directly available.
- Whether authentication or special headers are required.
- How the downloaded document should be named.

For example, suppose a new AMC has a page like:

```
https://example-amc.com/resources/factsheets
```

The developer should first determine how the latest factsheet can be identified before writing the automation.

# 6.3 Step 1 — Create a New Site Module

Navigate to:

```
sites/
```

Create a new Python file using the AMC's name.

For example:

```
sites/
├── hdfc.py
├── sbi.py
├── axis.py
├── tata.py
└── new_amc.py
```

The filename should be:

- Short and descriptive.
- Written in lowercase.
- Based on the AMC name.
- Free from unnecessary spaces or special characters.

For example:

```
new_amc.py
```

---

# 6.4 Step 2 — Create the Main Download Function

Each AMC module should expose a main function that can be called by the central orchestrator.

For example:

```
def download_new_amc():
    pass
```

The function acts as the entry point for the AMC-specific automation.

A basic structure could look like:

```
def download_new_amc():

    print("Starting New AMC factsheet download...")

    # Website navigation

    # Locate latest factsheet

    # Extract PDF URL

    # Download document

    # Save document

    print("New AMC factsheet download completed.")
```

The exact implementation depends on how the target AMC website works.

---

# 6.5 Step 3 — Implement Website-Specific Logic

The next step is to implement the logic required to find the latest factsheet.

For example, if the website contains a direct PDF link:

```
<a href="https://example-amc.com/files/factsheet-july-2026.pdf">
    Download Factsheet
</a>
```

The automation needs to identify that link.

Depending on the website, this may be done using:

- Playwright
- Selenium
- HTML selectors
- API requests
- Page source parsing
- URL extraction utilities

For example, using Playwright:

```
from playwright.sync_api import sync_playwright


def download_new_amc():

    with sync_playwright() as p:

        browser = p.chromium.launch()

        page = browser.new_page()

        page.goto(
            "https://example-amc.com/resources/factsheets"
        )

        pdf_url = page.locator(
            "a[href$='.pdf']"
        ).first.get_attribute("href")

        print("Factsheet URL:", pdf_url)

        browser.close()
```

This is only an example. The selector and navigation logic must be adapted to the actual AMC website.

---

# 6.6 Step 4 — Use Existing Shared Utilities

Before implementing download functionality manually, check the project's shared utilities.

For example:

```
download_utils.py
extract_urls.py
config.py
```

If `download_utils.py` already provides the required download functionality, reuse it.

For example:

```
from download_utils import download_file
```

Then:

```
download_file(
    pdf_url,
    output_path
)
```

This is preferred over creating another download implementation inside `new_amc.py`.

The responsibility should remain separated:

```
new_amc.py
    │
    ├── Navigate website
    ├── Find factsheet
    └── Identify PDF URL
                │
                ▼
       download_utils.py
                │
                └── Download PDF
```

This keeps the new module consistent with the rest of the project.

---

# 6.7 Step 5 — Handle Website-Specific Conditions

Different AMC websites may behave differently.

For example:

- The factsheet may be hidden behind a button.
- The page may require scrolling.
- The latest document may appear first.
- The URL may be generated dynamically.
- The website may use an API.
- The PDF may not have a `.pdf` extension.
- Multiple factsheets may be displayed.

Therefore, the new module should contain only the logic necessary for that AMC.

For example:

```
def download_new_amc():

    try:

        # Open website

        # Navigate to factsheet section

        # Identify latest factsheet

        # Extract PDF URL

        # Download PDF

        print("New AMC completed successfully.")

    except Exception as error:

        print(
            f"New AMC download failed: {error}"
        )
```

The error handling should follow the existing project's conventions.

---

# 6.8 Step 6 — Test the New Module Independently

Before connecting the new module to `main.py`, test it independently.

For example:

```
if __name__ == "__main__":
    download_new_amc()
```

This allows the developer to run:

```
python sites/new_amc.py
```

and verify that:

1. The website opens correctly.
2. The correct page is reached.
3. The latest factsheet is identified.
4. The correct PDF URL is extracted.
5. The PDF downloads successfully.
6. The file is saved in the expected location.
7. Errors are handled correctly.

Testing independently makes debugging significantly easier.

---

# 6.9 Step 7 — Import the Function into `main.py`

Once the new AMC module is working correctly, connect it to the central orchestrator.

Open:

```
main.py
```

Add the required import.

For example:

```
from sites.new_amc import download_new_amc
```

The exact import style should follow the existing imports in `main.py`.

For example, if the project currently uses:

```
from sites.hdfc import download_hdfc
from sites.axis import download_axis
from sites.tata import download_tata
```

then the new function should follow the same pattern:

```
from sites.new_amc import download_new_amc
```

# 6.10 Step 8 — Add the Function to `run_all()`

The final integration step is to add the new function to the `run_all()` workflow.

For example:

```
def run_all():

    download_hdfc()

    download_axis()

    download_tata()

    download_new_amc()
```

Now the new AMC becomes part of the complete automation process.

When the main application is executed:

```
python main.py
```

the new AMC will be processed along with the existing AMC modules.

---

# 6.11 How the Complete Integration Works

After completing all the steps, the structure will look like:

```
main.py
   │
   ├── download_hdfc()
   ├── download_axis()
   ├── download_tata()
   └── download_new_amc()
                 │
                 ▼
        sites/new_amc.py
                 │
                 ├── Open website
                 ├── Navigate to factsheet
                 ├── Find latest document
                 ├── Extract URL
                 │
                 ▼
        download_utils.py
                 │
                 ▼
          factsheets/
                 │
                 └── New AMC PDF
```

This demonstrates the separation between the **orchestrator**, **AMC-specific logic**, and **shared utilities**.

---

# 6.12 Recommended Development Checklist

When adding a new AMC, use the following checklist.

### Website Analysis

- [ ]  Identify the factsheet/resource page.
- [ ]  Identify how the latest factsheet is determined.
- [ ]  Check whether the website requires JavaScript.
- [ ]  Check whether an API is used.
- [ ]  Identify the PDF/download URL.

### Development

- [ ]  Create a new file inside `sites/`.
- [ ]  Create the `download_new_amc()` function.
- [ ]  Implement the website-specific navigation.
- [ ]  Extract the required factsheet URL.
- [ ]  Reuse existing shared utilities.
- [ ]  Use the configured output location.
- [ ]  Add appropriate error handling.

### Testing

- [ ]  Run the new AMC module independently.
- [ ]  Verify the correct factsheet is selected.
- [ ]  Verify the PDF downloads successfully.
- [ ]  Verify the filename and location.
- [ ]  Check error scenarios.

### Integration

- [ ]  Import the function into `main.py`.
- [ ]  Add the function to `run_all()`.
- [ ]  Run the complete automation.
- [ ]  Verify that existing AMC modules still work.

---

# 6.13 Important Development Principle

The most important principle when adding a new AMC is:

> **Add the new website-specific logic without unnecessarily modifying existing AMC modules or shared functionality.**

For example, if a new AMC requires a completely different website structure, the preferred approach is:

```
sites/new_amc.py
```

rather than changing:

```
sites/hdfc.py
sites/axis.py
sites/tata.py
```

Similarly, if the new AMC requires functionality that is genuinely common to multiple modules, consider extending the appropriate shared utility instead of creating duplicate code.

This keeps the project modular and makes future maintenance easier.

---

# 6.14 Example Final Module

A simplified final implementation could look like:

```
from download_utils import download_file
from config import FACTSHEET_DIR


def download_new_amc():

    try:

        # 1. Website-specific logic
        pdf_url = find_latest_factsheet_url()

        if not pdf_url:
            print("Factsheet URL not found.")
            return

        # 2. Define output path
        output_path = (
            f"{FACTSHEET_DIR}/new_amc_factsheet.pdf"
        )

        # 3. Use shared download utility
        download_file(
            pdf_url,
            output_path
        )

        print(
            "New AMC factsheet downloaded successfully."
        )

    except Exception as error:

        print(
            f"New AMC download failed: {error}"
        )


if __name__ == "__main__":
    download_new_amc()
```

The actual implementation will vary depending on the target website, but the overall structure should remain consistent with the project's architecture.

---

# 6.15 Summary

Adding a new fund manager should be a controlled and repeatable process.

The developer does not need to modify the entire automation system. Instead, the new AMC is implemented as an independent module and then connected to the existing orchestrator.

The complete process is:

```
1. Analyze AMC Website
        ↓
2. Create sites/new_amc.py
        ↓
3. Define download_new_amc()
        ↓
4. Implement Website-Specific Logic
        ↓
5. Reuse Shared Utilities
        ↓
6. Test Independently
        ↓
7. Import into main.py
        ↓
8. Add to run_all()
        ↓
9. Test Complete Automation
```

This modular approach ensures that the automation system can continue to grow as new fund managers need to be supported, while minimizing the impact on existing integrations.