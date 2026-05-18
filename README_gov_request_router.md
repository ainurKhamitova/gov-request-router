# 🏛️ Government Citizen Request Router

> An AI-powered automation workflow that receives citizen service requests, classifies them using a local LLM, routes them to the correct government department, notifies stakeholders, and logs everything — with zero manual intervention.

---

## 📌 Overview

Government service desks receive hundreds of citizen requests daily across multiple departments. This workflow automates the entire triage process — from intake to department notification — using a fully local AI model, ensuring citizen data never leaves the government network.

---

## 🎯 Use Case

| Problem                                               | Solution                                               |
| ----------------------------------------------------- | ------------------------------------------------------ |
| Manual request classification is slow and error-prone | Ollama LLM classifies requests instantly               |
| Citizens wait days for acknowledgment                 | Automated email sent within seconds                    |
| No audit trail for incoming requests                  | Every request logged to Google Sheets                  |
| Staff waste time routing emails manually              | Switch node routes to correct department automatically |

---

## 🏗️ Architecture

```
Webhook (Citizen Form)
  └── Generate Ticket ID (Code)
        └── Classify Request + Priority (Ollama LLM Chain)
              └── Extract Category & Priority (Code)
                    └── Switch (Route by Department)
                          ├── Ministry of Infrastructure
                          ├── Ministry of Health
                          ├── Ministry of Social Services
                          ├── Ministry of Education
                          └── Other / General
                                └── Gmail → Citizen Acknowledgment
                                            └── Google Sheets → Audit Log
```

---

## ⚙️ Tech Stack

| Component           | Tool                      |
| ------------------- | ------------------------- |
| Workflow Automation | n8n                       |
| AI Classification   | Ollama (llama3.2) — local |
| Email Notifications | Gmail (OAuth2)            |
| Audit Logging       | Google Sheets             |
| Entry Point         | n8n Webhook               |

---

## 🔒 Privacy by Design

All AI inference runs locally via **Ollama**. Citizen request data never leaves the government server. No third-party AI APIs are used.

---

## 🚀 Setup & Installation

### Prerequisites

- n8n (Docker recommended)
- Ollama installed and running
- Gmail account with OAuth2 configured
- Google Sheets API enabled

### 1. Install Ollama & Pull Model

```bash
curl -fsSL https://ollama.ai/install.sh | sh
ollama pull llama3.2
```

### 2. Run n8n

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### 3. Import Workflow

1. Download `gov_request_router.json`
2. In n8n → **Add Workflow → Import from File**
3. Select the JSON file

### 4. Configure Credentials

- **Gmail:** Add OAuth2 credentials in n8n
- **Google Sheets:** Enable Drive + Sheets API in Google Cloud Console
- **Ollama:** Ensure it's running at `http://localhost:11434`

### 5. Set Up Google Sheet

Create a sheet called `Citizen Requests Log` with these headers:

| ticket_id | full_name | email | category | priority | department_name | message | submitted_at | status |
| --------- | --------- | ----- | -------- | -------- | --------------- | ------- | ------------ | ------ |

---

## 🧪 Testing

Send a test request via curl:

```bash
curl -X POST http://localhost:5678/webhook/citizen-request \
  -H "Content-Type: application/json" \
  -d '{
    "full_name": "Ahmed Al Mansoori",
    "email": "ahmed@example.com",
    "message": "There is a broken streetlight on Al Nahda Street for 2 weeks"
  }'
```

### Expected Output

- Ticket created: `TKT-1747234567890`
- Category: `INFRASTRUCTURE`
- Priority: `HIGH`
- Citizen receives acknowledgment email
- Ministry of Infrastructure receives notification
- Row added to Google Sheets with status `OPEN`

---

## 📊 Classification Categories

| Category          | Example Requests                             |
| ----------------- | -------------------------------------------- |
| `INFRASTRUCTURE`  | Road damage, streetlights, public facilities |
| `HEALTH`          | Medical services, health complaints          |
| `LICENSING`       | Trade licenses, permits, renewals            |
| `SOCIAL_SERVICES` | Social support, welfare, family services     |
| `EDUCATION`       | School enrollment, education complaints      |
| `OTHER`           | General inquiries, suggestions               |

## 🎯 Priority Rules

| Priority   | Criteria                                 |
| ---------- | ---------------------------------------- |
| `CRITICAL` | Immediate danger or public safety threat |
| `HIGH`     | Affects many people or long unresolved   |
| `MEDIUM`   | Standard service request                 |
| `LOW`      | General inquiry or suggestion            |

---

## 📁 Project Structure

```
gov-request-router/
├── gov_request_router.json    # n8n workflow export
└──README.md
```
