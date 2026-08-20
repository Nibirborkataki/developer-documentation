
# Troubleshooting & Maintenance


## ## 7.1 Overview

The Factsheet Download Automation system depends on multiple external AMC websites, browser automation, server configuration, and local file storage. Because of these dependencies, certain issues may occur when running the automation in a production or scheduled environment.

This chapter documents the major issues encountered during development and server deployment, along with their causes, solutions, and recommended troubleshooting approaches.

The main issues covered in this chapter are:

1. **Bot detection when running browser automation on the server**
2. **Changing CSS selectors and timeout errors**
3. **Storage continuously filling due to old factsheets**
4. Additional troubleshooting and maintenance cases can be added as they are encountered.

---

# 7.2 Running Browser Automation in Headless Mode

All AMC automation scripts are designed to run in **headless mode**.

Headless mode means that the browser runs without displaying a graphical browser window.

This is particularly important for the production server because the server does not have a monitor or a desktop environment where a developer can manually see the browser.

The normal execution is therefore designed to run without displaying the browser UI.

However, some AMC websites have **bot-detection mechanisms** that can identify automated browser activity.

This created a problem where an automation script could work correctly when tested manually but behave differently when executed on the server.

---

# 7.3 Bot Detection Issue

During server testing, some AMC websites detected the browser automation as a bot.

This was one of the major issues encountered while preparing the automation for scheduled execution through a cron job.

The problem can be represented as:

Cron Job

   ↓

Python Automation

   ↓

Browser Starts

   ↓

AMC Website

   ↓

Bot Detection

   ↓

Automation Blocked / Unexpected Behaviour

Since the server does not have a graphical display, simply running a normal browser with a visible interface is not a practical solution.

---

# 7.4 Solution: X Virtual Frame Buffer (Xvfb)

The issue was resolved by using **X Virtual Frame Buffer (Xvfb)**.

Xvfb provides a virtual display environment on the server.

In simple terms, it creates a **fake screen** in memory so that applications which expect a graphical display can run even though the server does not have a physical monitor.

The process can be understood as:

Server

   │

   ├── No Physical Monitor

   │

   ▼

Xvfb

   │

   ├── Creates Virtual Display

   │

   ▼

Browser Automation

   │

   ▼

AMC Website

The browser can therefore run inside this virtual display environment without requiring an actual monitor.

This allows us to maintain the server-based automation while providing the browser with a graphical environment when required.

---

# 7.5 Running the Automation with Xvfb

Instead of running the automation directly:

python3 main.py

the automation is executed through `xvfb-run`:

xvfb-run -a python3 main.py

### Command Explanation

xvfb-run -a python3 main.py

can be broken down as:

|Part|Purpose|
|---|---|
|`xvfb-run`|Runs a command inside a virtual X display|
|`-a`|Automatically selects an available display number|
|`python3`|Starts Python 3|
|`main.py`|Starts the main automation|

The `-a` option is particularly useful for scheduled execution because it allows Xvfb to automatically find an available display rather than requiring a fixed display number.

---

# 7.6 Why Xvfb Is Used Instead of Running a Visible Browser

The production server does not require a browser window that a user can see.

The objective is:

Headless Server Automation

          +

Virtual Display

          ↓

Stable Browser Execution

Xvfb provides the virtual display without requiring a physical GUI.

This makes it suitable for automated execution through a cron job.

The final scheduled execution can therefore run:

xvfb-run -a python3 main.py

instead of requiring a developer to manually open the browser.

---

# 7.7 Changing CSS Selectors and Website Elements

Another common issue is that AMC websites can change their HTML or CSS structure.

The automation relies on elements such as:

- Buttons
- Links
- Cards
- Download elements
- Factsheet containers
- Popup triggers
- CSS selectors

If an AMC redesigns its website, an element used by the automation may no longer exist in the expected form.

For example, the automation may previously depend on:

.download-button

while the AMC later changes the element to:

.download-report

The automation will continue searching for the old element and eventually fail.

---

# 7.8 Identifying a Selector Change

The most common error in this situation is a **timeout error**.

For example:

Running Bandhan AMC...

Clicking latest factsheet...

Error in Bandhan: Timeout 30000ms exceeded while waiting for event "popup"

  

=========================== logs ===========================

waiting for event "popup"

============================================================

The important part of this error is:

Timeout 30000ms exceeded

This means that the automation waited for the expected event or element for 30 seconds, but it never occurred.

In this particular example, the automation was waiting for:

popup

but the expected popup did not appear.

---

# 7.9 What Does a Timeout Mean?

A timeout **does not always mean that the website is down**.

It means that the automation was waiting for something to happen, but that condition was not satisfied within the configured timeout period.

Possible reasons include:

- The CSS selector has changed.
- The element has been removed.
- The button has been renamed.
- The website layout has changed.
- The expected popup no longer appears.
- The element is now located somewhere else.
- The page loads differently.
- A popup, banner, or other element is blocking the expected action.
- The website is responding slowly.

Therefore, when a timeout occurs, the first step should be to determine **what the automation was waiting for**.

---

# 7.10 Example of a Selector Failure

Suppose the original automation expects:

page.locator(".factsheet-button")

The AMC website later changes its HTML from:

<a class="factsheet-button">

    Download Factsheet

</a>

to:

<a class="download-document">

    Download Factsheet

</a>

The old selector:

.factsheet-button

no longer identifies the element.

The automation waits for the element until the timeout is reached.

The result can therefore be:

Timeout 30000ms exceeded

---

# 7.11 How to Troubleshoot a Timeout

When a timeout occurs:

### Step 1 — Identify the AMC

Look at the execution output:

Running Bandhan AMC...

This identifies which AMC failed.

### Step 2 — Read the Error

Determine what the automation was waiting for.

For example:

waiting for event "popup"

### Step 3 — Open the AMC Website

Open the relevant page manually in a browser.

### Step 4 — Check the Website

Verify whether:

- The factsheet button still exists.
- The expected popup still appears.
- The button location has changed.
- The HTML structure has changed.

### Step 5 — Inspect the Element

Use browser Developer Tools to inspect the current HTML and determine the correct selector or interaction.

### Step 6 — Update the AMC Module

Modify only the affected AMC script wherever possible.

### Step 7 — Test the AMC Independently

Run the affected script before running the entire automation.

### Step 8 — Run the Complete Automation

Once the individual AMC works correctly, execute `main.py` again.

---

# 7.12 Important: Do Not Increase the Timeout Immediately

When a timeout occurs, it may be tempting to simply increase:

30000

to:

60000

However, this should not be the first solution.

If the element has actually been removed or its selector has changed, increasing the timeout will not solve the problem.

For example:

Old selector

     ↓

Element no longer exists

     ↓

Wait 30 seconds

     ↓

Timeout

  

Increasing to 60 seconds

     ↓

Element still doesn't exist

     ↓

Timeout again

The correct approach is to investigate why the expected element or event is no longer occurring.

---

# 7.13 Storage Filling Due to Old Factsheets

Another issue encountered during server execution was that the available storage was being consumed quickly.

The automation downloads factsheets regularly, and if every new factsheet is retained indefinitely, the number of files continues to increase.

The process initially looked like:

New Factsheet

      ↓

Download

      ↓

Old Factsheet remains

      ↓

Next Factsheet

      ↓

Download

      ↓

Another Old Factsheet remains

      ↓

Storage keeps increasing

Over time, this could cause the server's disk space to become critically low.

---

# 7.14 Solution: Replace the Previous Factsheet

The storage issue was resolved by changing the file-management logic.

Instead of keeping multiple old versions of the same AMC's factsheet, the automation checks whether a **new factsheet is available**.

When a new PDF is identified:

Check Latest Factsheet

        ↓

Is it a new PDF?

        │

      Yes

        ↓

Download New PDF

        ↓

Delete Previous Factsheet

        ↓

Keep Latest Version

This ensures that only the required/latest factsheet remains available for each mutual fund.

---

# 7.15 Using Python `os` for File Management

The file-management logic uses Python's `os` module to work with files stored on the server.

The automation can use the filesystem to:

- Check whether a file exists.
- Identify files in a directory.
- Determine the existing factsheet.
- Remove an old factsheet.
- Maintain the required output directory.

For example:

import os

A file can be checked using:

if os.path.exists(file_path):

    print("File exists")

An old file can be removed using:

os.remove(file_path)

The actual implementation should ensure that the old file is deleted **only after the new factsheet has been successfully identified/downloaded**.

---

# 7.16 Why the Storage Solution Is Important

The automation runs regularly, so storage management is an important part of maintenance.

Without file cleanup:

Week 1 → 50 PDFs

Week 2 → 100 PDFs

Week 3 → 150 PDFs

Week 4 → 200 PDFs

...

The storage requirement continues to increase.

With the latest-file approach:

AMC A → Latest PDF

AMC B → Latest PDF

AMC C → Latest PDF

...

The storage requirement remains much more predictable.

This makes the server more suitable for long-running scheduled automation.

---

# 7.17 Current Maintenance Strategy

The current automation follows three important maintenance principles:

### 1. Virtual Display for Browser Execution

For AMC websites where bot detection or browser behaviour requires a display environment:

xvfb-run -a python3 main.py

is used.

### 2. AMC-Specific Selector Maintenance

If a timeout occurs, investigate the affected AMC website and update its selectors or interaction logic when the website structure changes.

### 3. Automatic File Cleanup

When a new factsheet becomes available, the previous factsheet for that AMC is removed so that unnecessary files do not continuously consume server storage.

---

# 7.18 Troubleshooting Quick Reference

|Issue|What It Usually Means|What to Check|
|---|---|---|
|Bot detection|Website detected automated browser behaviour|Run through Xvfb|
|`Timeout 30000ms exceeded`|Expected element/event did not occur|Check selector and website structure|
|`waiting for event "popup"`|Expected popup was not triggered|Check button/action and current website behaviour|
|Element not found|Website element/selector changed|Inspect current HTML|
|Storage filling quickly|Old PDFs are accumulating|Check file cleanup logic|
|Download not occurring|URL or download element may have changed|Verify extracted URL|
|One AMC fails|Usually AMC-specific issue|Investigate its `sites/` module|
|Multiple AMCs fail|Could indicate shared/infrastructure issue|Check utilities, server, network, and configuration|

---

# 7.19 Summary

The Factsheet Download Automation is designed to run continuously on a server, so troubleshooting and maintenance are important for reliable long-term execution.

Three major production issues have been identified and addressed:

### Bot Detection

Some AMC websites detect automated browsers. **Xvfb** provides a virtual display environment that allows the browser automation to run on a server without requiring a physical monitor.

xvfb-run -a python3 main.py

### Changing Website Elements

AMC websites can change their HTML/CSS structure. When an expected element or event is no longer available, the automation may produce errors such as:

Timeout 30000ms exceeded

The solution is to inspect the current website structure and update the affected AMC-specific automation logic.

### Storage Management

Regular factsheet downloads can cause storage to grow continuously if old files are retained. The automation now checks for a new PDF and, after successfully obtaining the latest factsheet, removes the previous factsheet for that AMC.

Together, these solutions make the automation more suitable for **long-running scheduled execution through cron jobs** and reduce the need for frequent manual intervention.