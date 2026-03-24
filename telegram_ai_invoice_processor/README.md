# Telegram AI Invoice Processor

An n8n workflow that accepts invoice files sent via Telegram (PDF, JPG, or PNG), extracts structured data using OCR + GPT-4.1, logs it to Google Sheets, and uploads the original file to Google Drive — then sends the user a confirmation message.

---

## How It Works

```
Telegram Message
      ↓
Download File (photo or document)
      ↓
Detect File Type (PDF / JPG / PNG)
      ↓
Switch Router
  ├── PNG  → OCR.space (Engine 2, isTable)
  ├── JPG  → OCR.space (Engine 2, isTable)
  ├── PDF  → OCR.space (Engine 2, isTable, filetype=PDF)
  └── Other → Send error message to user
      ↓
GPT-4.1 LLM Extract (structured JSON)
      ↓
Cleanup & Normalize JSON
      ↓
Append to Google Sheets
      ↓
Merge (binary file + invoice data)
      ↓
Upload to Google Drive
      ↓
Set Info (filename + invoice details)
      ↓
GPT-4.1 Invoice Processor (compose confirmation message)
      ↓
Send Confirmation to User via Telegram
```

---

## Features

- Accepts invoices sent as **photos** or **documents** via Telegram
- Supports **PDF, JPG, and PNG** formats with dedicated OCR configurations
- Uses **GPT-4.1** to intelligently extract and clean structured invoice data
- Handles OCR errors and corrects obvious spelling mistakes in item descriptions
- Logs all invoice fields to a **Google Sheets** database
- Saves original invoice files to **Google Drive** with timestamped filenames
- Sends a clean, formatted **confirmation message** back to the user
- Graceful fallback message for unsupported file types

---

## Extracted Invoice Fields

| Field | Description |
|---|---|
| Invoice Number | Unique invoice identifier |
| Invoice Date | Date the invoice was issued |
| Due Date | Payment due date |
| Delivery Date | Expected or actual delivery date |
| Vendor Name | Name of the issuing company |
| Customer Name | Name of the recipient |
| Customer Address | Full address of the recipient |
| Currency | Detected currency (USD, EUR, GBP, etc.) |
| Subtotal | Pre-tax total |
| Tax | Tax amount |
| Total | Final payable amount |
| Item Descriptions | Line item descriptions (newline-separated) |
| Item Quantities | Per-item quantities |
| Item Unit Prices | Per-item unit prices |
| Item Amounts | Per-item totals |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation |
| [Telegram Bot API](https://core.telegram.org/bots/api) | Message & file handling |
| [OCR.space API](https://ocr.space/ocrapi) | PDF and image OCR |
| [OpenAI GPT-4.1](https://platform.openai.com) | Invoice data extraction & message generation |
| [Google Sheets API](https://developers.google.com/sheets) | Invoice database logging |
| [Google Drive API](https://developers.google.com/drive) | File storage |

---

## Setup Instructions

### 1. Prerequisites
- n8n instance (self-hosted or cloud)
- Telegram Bot Token — create one via [@BotFather](https://t.me/botfather)
- OCR.space API key — free tier available at [ocr.space](https://ocr.space/ocrapi)
- OpenAI API key with access to `gpt-4.1`
- Google account with Sheets and Drive API enabled

### 2. Import the Workflow
1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `workflow.json` from this folder
4. Save the workflow

### 3. Configure Credentials
Set up the following credentials in n8n:

| Credential | Type | Used By |
|---|---|---|
| Telegram Bot | Telegram API | Trigger, download, send response |
| OCR.space Key | HTTP Header Auth (`apikey: YOUR_KEY`) | analyze_pdf, analyze_jpg, analyze_png |
| OpenAI | OpenAI API | brain (GPT-4.1) |
| Google Sheets | OAuth2 | append_invoice_details |
| Google Drive | OAuth2 | upload_invoice |

### 4. Update Placeholders
Search the workflow JSON for these placeholders and replace with your own values:

| Placeholder | Replace With |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Your Google Sheets document ID |
| `YOUR_GOOGLE_SHEET_LINK` | Full URL to your sheet |
| `YOUR_GOOGLE_DRIVE_ID` | Your Google Drive folder ID |

### 5. Google Sheets Setup
Create a sheet named **Sheet1** with these exact column headers in row 1:

```
Invoice Number | Invoice Date | Due Date | Delivery Date | Vendor Name | Customer Name | Customer Address | Currency | Subtotal | Tax | Total | Item Descriptions | Item Quantities | Item Unit Prices | Item Amounts
```

### 6. Activate
Toggle the workflow to **Active** in n8n. Your Telegram bot is now live.

---

## Usage

Send any of the following to your Telegram bot:

- A **photo** of an invoice (JPG or PNG)
- A **PDF** invoice sent as a document
- A **JPG or PNG** invoice sent as a document (with "Send as document" checked)

The bot will reply with a confirmation summary including the invoice number, total, due date, and a link to your invoice database.

---

## File Naming Convention in Google Drive

Uploaded files are named automatically:

```
Invoice {InvoiceNumber} [DD-Month-YYYY | HH:MM AM/PM]
```

Example: `Invoice INV-2026-0041 [23-March-2026 | 05:42 PM]`

---

## Notes

- Telegram compresses photos when sent normally — for best OCR accuracy, use **Send as document** to preserve original resolution
- OCR Engine 2 is used for all file types for superior table and layout recognition
- The workflow uses `combineByPosition` merge — both branches (invoice data + binary file) must produce exactly 1 item each
- `retryOnFail` is enabled on OCR, LLM, and Drive nodes to handle transient API errors

---

## License

MIT — free to use, modify, and distribute.
