# Serper Lead Scraper

An n8n workflow that takes a target audience description from a form, generates smart search queries using GPT, scrapes business websites via Serper + ScraperAPI, enriches each lead with contact data, scores their speed-to-lead readiness, upserts results into Google Sheets, and sends an email summary.

---

## Workflow Canvas

![Serper Lead Scraper Workflow](./workflow-canvas.png)

---

## How It Works

```
n8n Form Submission (industry, location, seniority, company size)
      ↓
LLM generates targeted search queries (GPT)
      ↓
Format queries for Serper API
      ↓
Search with Serper → raw organic results
      ↓
Split results → Loop over each lead
      ↓
┌─────────────────────────────────────┐
│  Serper Router (Contact + Trustpilot URLs)  │
│  Fetch Homepage (ScraperAPI)               │
│  Fetch Contact Page (ScraperAPI)           │
│  Clean & Parse HTML                        │
│  Detect + Extract enrichment data          │
└─────────────────────────────────────┘
      ↓
Merge Lead Data
      ↓
Score Speed-to-Lead (0–10)
      ↓
Cleanup Fields
      ↓
Upsert to Google Sheets
      ↓
Summarize batch
      ↓
Send Email Notification (Gmail)
      ↓
Update Sheet with final status
```

---

## Features

- **n8n Form trigger** — user inputs industry, country, state, seniority level, company size, number of search terms, and top results count
- **LLM-powered query generation** — GPT creates targeted search strings based on the input context
- **Serper API search** — retrieves organic Google results for each generated query
- **ScraperAPI web scraping** — fetches homepage and contact page HTML for each lead
- **Trustpilot + contact page routing** — Serper Router finds additional URLs per domain
- **AI enrichment extraction** — detects emails, phone numbers, chat widgets, booking schedulers, instant quote tools, forms, and call-to-action buttons
- **Speed-to-lead scoring** — scores each lead 0–10 based on their contact readiness gaps
- **Google Sheets upsert** — inserts new leads or updates existing ones by domain
- **Gmail notification** — sends a summary email after each batch completes
- **Loop architecture** — processes leads one at a time with a Wait node to respect rate limits

---

## Speed-to-Lead Scoring System

Each lead is scored based on what contact/conversion tools they have on their website:

| Signal | Points |
|---|---|
| Booking / scheduler present | +4 |
| Contact form present | +3 |
| Chat widget present | +3 |
| Call-Now CTA | +2 |
| Instant quote / estimate tool | +2 |
| Email address visible | +1 |
| Phone number detected | +1 |
| **Maximum score** | **10** |

Leads with low scores have the most gaps — meaning the highest opportunity for outreach.

---

## Form Inputs

| Field | Type | Description |
|---|---|---|
| Seniority Level | Checkbox | CEO, Founder, Owner |
| Company Size | Dropdown | 1–10, 11–20, 21–30 |
| Industry | Text | Target industry (e.g. "roofing", "dental clinics") |
| Country | Text | Target country |
| State | Text | Target state or region |
| num_search_terms | Number | How many search queries to generate |
| top_results | Number | How many results to process per query |

---

## Enriched Lead Fields (Google Sheets Output)

| Field | Description |
|---|---|
| Domain | Company website domain |
| Company Name | Extracted from page or search result |
| Email | Email found on homepage or contact page |
| Phone | Phone number detected |
| Contact URL | URL of the contact page |
| Has Chat Widget | Boolean — chat tool detected |
| Has Scheduler | Boolean — booking tool detected |
| Has Instant Quote | Boolean — quote tool detected |
| Has Call-Now CTA | Boolean — prominent call CTA |
| Has Form | Boolean — contact form present |
| Speed-to-Lead Score | 0–10 numeric score |
| Pain Point Summary | AI-generated gap analysis |
| Gaps | List of missing conversion signals |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| [n8n](https://n8n.io) | Workflow automation |
| [Serper API](https://serper.dev) | Google Search results |
| [ScraperAPI](https://www.scraperapi.com) | Website HTML scraping |
| [OpenAI GPT](https://platform.openai.com) | Query generation + data extraction |
| [Google Sheets](https://developers.google.com/sheets) | Lead database (upsert) |
| [Gmail](https://developers.google.com/gmail) | Email batch notifications |

---

## Setup Instructions

### 1. Prerequisites
- n8n instance (self-hosted or cloud)
- Serper API key — get one at [serper.dev](https://serper.dev)
- ScraperAPI key — get one at [scraperapi.com](https://www.scraperapi.com)
- OpenAI API key
- Google account with Sheets and Gmail API enabled

### 2. Import the Workflow
1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `workflow.json`
4. Save

### 3. Configure Credentials

| Credential | Type | Used By |
|---|---|---|
| Serper API key | HTTP Header Auth (`X-API-KEY`) | searchwithSerper, Serper Router |
| ScraperAPI key | Query param (`api_key`) | Fetch Homepage, Fetch Contact |
| OpenAI | OpenAI API | OpenAI Chat Model |
| Google Sheets | OAuth2 | Upsert_leads, update sheet |
| Gmail | OAuth2 | Send a message |

### 4. Google Sheets Setup
Create a sheet with these column headers in row 1 — the upsert node matches on **Domain**:

```
Domain | Company Name | Email | Phone | Contact URL | Has Chat Widget | Has Scheduler | Has Instant Quote | Has Call-Now CTA | Has Form | Speed-to-Lead Score | Pain Point Summary | Gaps
```

### 5. Update the notification email
In the `Send a message` node, update the `sendTo` field to your own email address.

### 6. Activate & Use
- Activate the workflow in n8n
- Open the form URL (shown in the `On form submission` node)
- Fill in your target audience details and submit
- Watch leads populate in your Google Sheet

---

## Notes

- The **Wait node** adds a delay between lead processing loops to avoid hitting ScraperAPI rate limits — adjust the duration based on your plan
- The **Upsert** uses domain as the unique key — re-running on the same industry will update existing rows rather than duplicate them
- ScraperAPI handles JavaScript-rendered pages — useful for sites that block simple HTTP requests
- Scores are capped at 10 regardless of how many signals are present

---

## License

MIT — free to use, modify, and distribute.
