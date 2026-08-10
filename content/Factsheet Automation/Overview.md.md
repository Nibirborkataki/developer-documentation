# Factsheet Download Automation

## Overview

The Factsheet Download Automation project is a Python-based automation system developed to automatically collect and download the latest mutual fund factsheets from different Asset Management Company (AMC) websites.

The automation reduces the need for manually visiting multiple websites, locating the latest factsheet, and downloading the corresponding PDF.

## Objective

The main objectives of the automation are:

- Automatically access AMC websites.
- Locate the latest available factsheet.
- Extract the factsheet PDF URL.
- Download the PDF automatically.
- Maintain a consistent file naming and storage process.
- Reduce manual effort and processing time.
- Handle website-specific differences and errors.

## Technologies Used

- Python
- Playwright
- Selenium
- Web Scraping
- HTML/CSS Selectors
- File Handling
- PDF Processing

## Supported AMC Websites

The automation has been implemented for multiple mutual fund websites, including:

- Axis Mutual Fund
- TATA Mutual Fund
- WhiteOak Capital
- Trust Mutual Fund
- Unifi Mutual Fund
- Baroda BNP Paribas Mutual Fund
- Taurus Mutual Fund
- And other AMC websites

## General Workflow

The overall process follows these steps:

1. Open the AMC website.
2. Navigate to the factsheet or resource section.
3. Identify the latest available factsheet.
4. Extract the PDF/download URL.
5. Download the factsheet.
6. Save the downloaded file in the required location.
7. Handle website-specific conditions or errors.
8. Verify the downloaded file.

## Project Structure

The documentation is organized into separate sections covering:

- Project overview
- Technologies and dependencies
- Automation workflow
- Website-specific implementations
- Error handling
- Troubleshooting
- Maintenance and future improvements