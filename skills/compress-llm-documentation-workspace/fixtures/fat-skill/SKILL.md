---
name: invoice-processor
description: Processes invoices.
---

# Invoice Processor

## Overview

This skill helps you process invoices. Invoices are commercial documents issued by a seller to a
buyer relating to a sale transaction and indicating the products, quantities, and agreed prices for
products or services the seller had provided the buyer. An invoice typically contains a header with
the seller and buyer details, a table of line items, and a total.

This skill was created to make invoice processing easier and more reliable. Before this skill
existed, people had to process invoices manually, which was slow and error prone. Now they do not
have to do that any more, which is a big improvement.

## Background on PDF

Most invoices arrive as PDF files. PDF stands for Portable Document Format, a file format developed
by Adobe in 1992 to present documents including text formatting and images in a manner independent
of application software, hardware, and operating systems. PDFs can contain text layers or be pure
scans of paper.

If a PDF has a text layer you can extract the text directly. If it does not, you will need OCR,
which stands for Optical Character Recognition — a technology that converts images of text into
machine-encoded text.

## When to Use This Skill

Use this skill when you need to process an invoice. You might want to do this if someone gives you
an invoice and asks you to extract data from it, or if you have a folder of invoices that need to
be turned into a spreadsheet, or in various other situations where invoices are involved.

## Setup

Before you start, make sure you have Python installed. You will need version 3.10 or later. You
should also install the dependencies. Run this command:

```bash
pip install -r requirements.txt
```

You may also want to set up a virtual environment first, which is generally considered a best
practice in the Python community. You can do this with `python -m venv .venv` and then activating
it. On macOS and Linux you activate it with `source .venv/bin/activate`, and on Windows you use
`.venv\Scripts\activate` instead.

## The Workflow

### Step 1: Identify the invoice type

First you should look at the invoice and figure out what kind it is. There are several kinds. Some
invoices are from our standard vendors and have a predictable layout. Others are one-offs. You can
usually tell by looking at the header.

### Step 2: Extract the text

Run the extraction script:

```bash
python scripts/extract.py INPUT.pdf --out extracted.json
```

This will produce a JSON file with the extracted text. If the PDF has no text layer the script will
fall back to OCR automatically, though this is slower and less accurate, so it is worth checking
the output carefully.

### Step 3: Parse the line items

Run:

```bash
python scripts/parse_items.py extracted.json --out items.csv
```

Line items have a code, a description, a quantity, a unit price, and a line total. The line total
should equal quantity times unit price. If it does not, something is wrong and you should stop and
flag it rather than guessing.

### Step 4: Validate

Run `python scripts/validate.py items.csv`. This checks that the line totals sum to the invoice
total, that the tax rate is one of the permitted values, and that the currency code is valid ISO
4217.

The validator exits non-zero if anything fails. Never submit an invoice that fails validation.

### Step 5: Submit

Run `python scripts/submit.py items.csv --vendor VENDOR_ID`. You need the vendor ID, which you can
look up in the vendor table.

## Output Format

The final output must be exactly this shape, because the downstream ERP import is positional:

```csv
vendor_id,invoice_number,line_code,description,quantity,unit_price,line_total,currency
```

## Tax Rates

The permitted tax rates are 0%, 5%, 12%, and 20%. Anything else is a data entry error and must be
escalated to finance rather than corrected locally.

## Currency Handling

All amounts are stored as integer minor units. Never use floating point for money — you will get
rounding errors that are impossible to reconcile later.

## Common Problems

Sometimes the OCR gets confused by handwritten annotations in the margins. If the extracted text
looks like nonsense, check whether the original was a scan.

Sometimes vendors send the same invoice twice. The submit script checks for duplicates by invoice
number, but only within the same vendor, so a duplicate from a different vendor ID will get
through.

Sometimes the invoice total includes shipping as a separate line and sometimes it is baked into the
line items. Both are fine. What is not fine is shipping appearing in both places, which happens
occasionally with one particular vendor and always needs a human.

## Tips

- Be careful and thorough.
- Double-check your work.
- If something seems off, it probably is.
- Read the invoice properly before starting.
- Take your time.

## Examples

### Example 1

Input: a standard invoice from Acme Supplies with 3 line items.
Output: 3 rows in the CSV.

### Example 2

Input: a standard invoice from Acme Supplies with 5 line items.
Output: 5 rows in the CSV.

### Example 3

Input: a standard invoice from Acme Supplies with 1 line item.
Output: 1 row in the CSV.

### Example 4

Input: an invoice where the line totals do not sum to the invoice total.
Output: stop, do not submit, escalate to finance with the discrepancy amount.

## Further Reading

See the docs.

Also `reference/vendor_layouts.md`.
