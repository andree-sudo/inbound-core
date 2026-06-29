# Inbound Core — n8n Workflow

🇸🇰 [Slovenčina](README.sk.md) · 🇺🇦 [Українська](README.uk.md)

---

### Overview

**Inbound Core** is an n8n automation workflow that processes incoming emails, evaluates their business relevance using an AI model (Google Gemini), and — for relevant messages — automatically creates a business proposal document, saves contact and pricing data to Google Sheets, and logs the result.

---

### How It Works

```
Gmail Trigger
     │
     ▼
Google Gemini (AI Intake Assistant)
     │
     ▼
Code in JavaScript1 (parse AI response)
     │
     ▼
If — Relevant?
 ├── NO  → Send alert email to admin
 └── YES → Create Google Doc (Proposal)
               │
               ▼
           Update Google Doc (fill in proposal details)
               │
               ▼
           Code in JavaScript (extract doc_id)
               │
               ▼
           Append / Update row in Google Sheets (log)
               │
               ▼
           Update Google Doc (add final amount)
```

---

### Nodes Description

| Node | Type | Purpose |
|------|------|---------|
| **Gmail Trigger** | Trigger | Polls inbox every minute for new emails |
| **Message a model** | Google Gemini | Analyzes the email: assesses relevance, extracts contact data, determines service category, fetches pricing |
| **Code in JavaScript1** | Code | Parses the AI JSON response; flags non-relevant emails |
| **If** | Condition | Routes the flow: relevant vs. not relevant |
| **Send a message1** | Gmail | Sends an admin alert if the email cannot be processed |
| **Create a document** | Google Docs | Creates a new proposal document named `Proposal — {Client Name}` |
| **Update a document** | Google Docs | Inserts proposal content (client info, service, base price) |
| **Code in JavaScript** | Code | Extracts the newly created document ID for further updates |
| **Append or update row in sheet** | Google Sheets | Logs the calculated result and processing details (Sheet 3) |
| **Update a document1** | Google Docs | Appends the final amount and calculation date to the proposal |
| **Google_Sheets_Add_Contact** | AI Tool | Saves contact record to Sheet 1 |
| **Google_Sheets_Add_Calculation_Input** | AI Tool | Saves calculation inputs to Sheet 2 |
| **Google_Sheets_Get_Pricing** | AI Tool | Reads pricing data for the detected service category |

---

### Supported Service Categories

| Service | category_id |
|---------|-------------|
| Cloud Migration | `srv_cloud_mig` |
| Data Pipeline Setup | `srv_data_eng` |
| AI Architecture Audit | `srv_ai_audit` |
| Custom Backend Development | `srv_custom_dev` |
| Legacy System Support | `srv_legacy_supp` |

---

### Email Relevance Rules

**Relevant** — the sender:
- Requests a cost estimate or consultation
- Wants to order a service
- Discusses a new project
- Expresses interest in any of the supported services listed above

**Not relevant** — the email is:
- Spam or marketing
- A job application or resume
- A personal message
- An automated system notification

---

### Prerequisites & Configuration

Before activating the workflow, replace all placeholder values in the JSON:

| Placeholder | What to put |
|-------------|-------------|
| `YOUR_CREDENTIAL_ID` | Your n8n credential ID for each service |
| `YOUR_SPREADSHEET_ID` | Google Sheets spreadsheet ID |
| `YOUR_SPREADSHEET_URL` | Full URL of the spreadsheet |
| `YOUR_FOLDER_ID` | Google Drive folder ID for proposals |
| `YOUR_EMAIL` | Admin email address for alert notifications |
| `YOUR_WEBHOOK_ID` | n8n webhook ID (if used) |
| `YOUR_INSTANCE_ID` | Your n8n instance ID |

**Required credentials in n8n:**
- Google Gemini (PaLM) API
- Google Sheets OAuth2
- Google Docs OAuth2
- Gmail OAuth2

**Google Sheets structure:**
- **Sheet 1 (`Аркуш1`)** — Contacts (`user_id`, `full_name`, `email`, `phone_number`, `status`)
- **Sheet 2** — Calculation inputs (`category_id`, `input_value_1`, `input_value_2`, `raw_email_body`)
- **Sheet 3 (`Аркуш3`)** — Processing log (`calculated_result`, `created_at`, `processing_log`)

---

### Notes

- The workflow is set to **inactive** by default — activate it manually after configuration.
- The Gmail Trigger polls **every minute**; adjust the frequency as needed.
- If the AI model response cannot be parsed (invalid JSON), the flow will fail silently — consider adding an error handler node.
- Phone numbers are stored **without** the leading `+` sign.
- If no name is found in the email, the contact is saved as `Unknown`.
