#  🤖 Smart-Lead-Intake-Bot


An automated, intelligent pipeline built with **n8n** to capture, normalize, classify, and store leads from multiple channels. This system leverages AI for intent scoring and implements strict data deduplication to ensure a clean, actionable database for the sales team.

## 🚀 Key Features

- **Multi-Channel Ingestion:** Captures leads seamlessly from website forms (Webhooks) and direct inquiries (Gmail).
- **Data Normalization:** Unifies diverse data payloads into a strict, single schema.
- **Zero Duplicate Rows:** Prevents CRM clutter by querying the database for existing leads within a 30-day window before insertion.
- **AI-Powered Classification:** Uses LLMs to analyze the lead's message, score their intent (1-5), and categorize the requested service.
- **Dual-Storage System:** Stores the official record in **PostgreSQL** while keeping a highly accessible copy in **Google Sheets** for the sales team.
- **Real-Time Alerts:** Sends instant, formatted summaries to the sales team via **Telegram**.

---

## 🔄 Workflow Architecture (Node by Node)

### 1. Triggers (Entry Points)
- **Webhook Node:** The primary entry point for leads coming from the website's contact form via a `POST` request.
- **Gmail Trigger:** Monitors a designated inbox for new inquiry emails, triggering parallel execution.
*Note: Each channel has its own independent initial path due to payload structure differences.*

### 2. Data Normalization
- **Edit Fields (Webhook Path):** Maps raw form data into a unified schema: `name`, `email`, `phone`, `company`, `service_interest`, `message`, `source`, and `received_at`.
- **Split Out:** Deconstructs any arrays or batched data from the form into individual items for easier processing.
- **Edit Fields (Email Path):** Transforms email metadata (subject, body, sender) into the exact same unified schema.

### 3. Merge
- **Merge Node:** Converges the Webhook and Email paths using the `Append` mode. From this point forward, every lead flows through a single stream with an identical data structure, regardless of its original source.

### 4. Custom Data Processing
- **Code (Python):** Executes custom Python logic to handle transformations that standard nodes cannot, such as formatting phone numbers, extracting email domains to identify companies, and standardizing the `received_at` timestamp.

### 5. Deduplication (Data Integrity)
- **Postgres Node (SQL Query):** Runs a query against the database: *Has this email been registered in the last 30 days?* Returns a boolean/row count.
- **IF Node:** Routes the workflow based on the SQL result:
  - `True` (Duplicate Found): The lead is stopped immediately (`Duplicate lead - stopped`). No storage, no notifications.
  - `False` (New Lead): The workflow proceeds to classification.

### 6. AI Classification
- **Message a Model (LLM):** Sends the normalized lead data to an AI model to extract and return:
  - `service_category`: The specific service the lead is interested in.
  - `intent_score`: A scale from 1 to 5 indicating how serious the lead is.
  - `language`: Detected language (`AR` or `EN`).
  - `summary`: A concise, one-line summary of the lead's request.

### 7. Dual Storage Execution
*The workflow branches to store data simultaneously in two locations:*
- **Postgres Node (Insert Row):** Saves the complete record (raw data + AI classification) into the PostgreSQL database as the single source of truth.
- **Google Sheets Node (Append Row):** Adds the same record to a spreadsheet, providing a frictionless interface for the sales team to view and manage leads.

### 8. Sales Notification
- **Telegram Node:** Pushes a real-time notification to the sales channel. The message includes a quick breakdown: Source, Service Category, Intent Score, and the one-line Summary.
- 
🤖**Telegram Notification Integration:**
  Created a dedicated bot via BotFather to obtain the API Token (TELEGRAM_BOT_TOKEN).


  Retrieved the target destination ID (TELEGRAM_CHAT_ID) using @userinfobot to ensure direct routing of alerts.
 

  Configured the n8n Telegram Node to securely send real-time structured summaries (Source, Intent Score, and Lead Summary) upon successful lead insertion.



---

**workflow n8n:**
Webhook ──► Edit Fields ──► Split Out ──┐
                                         ├──► Merge ──► Code in Python ──► Execute a SQL query ──► If
Gmail Trigger ──► Edit Fields1 ──────────┘                                                          │
                                                                                    ┌── true ────────┴── Duplicate lead - stopped
                                                                                    │
                                                                                    └── false ──► Message a model ──┬──► Insert rows in a table
                                                                                                                                                  └──► Append row in sheet ──► Send a text message


## 🛠️ Tech Stack

- **Automation Engine:** [n8n](https://n8n.io/)
- **Database:** PostgreSQL
- **AI / LLM:** Gemini (via n8n integration)
- **Data Processing:** Python
- **Integrations:** Google Sheets API, Gmail API, Telegram Bot API

---
### 🧪 Testing
For detailed verification logs and test results, please refer to: [TESTING.md](TESTING.md)

## 📸 Workflow Visualization

Here is the visual overview of the Smart Lead Intake Bot pipeline in n8n:
Here is the complete visual overview of the Smart Lead Intake Bot pipeline in n8n:

![n8n Workflow Diagram](https://github.com/Marahamj/Smart-Lead-Intake-Bot/blob/main/image.png?raw=true)












