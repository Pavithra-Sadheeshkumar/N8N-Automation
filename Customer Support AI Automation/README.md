# 🤖 Customer Support AI Automation

An intelligent **n8n workflow** designed to triage, route, and resolve customer support inquiries using **GPT-5-mini** and **RAG (Retrieval-Augmented Generation)**. 

---

## ✨ Key Features

* **AI Sentiment & Intent Analysis**: Automatically classifies emails into categories like `refund`, `complaint`, or `technical_issue`.
* **Urgency-Based Routing**: Detects angry sentiment or "ASAP" language to flag tickets as **Urgent** and trigger immediate Slack alerts.
* **Knowledge-Base Integration (RAG)**: Uses a **Supabase Vector Database** to "read" the company docs and provide factual, non-hallucinated answers.
* **Automated CRM Management**: 
    * Searches for existing contacts in **HubSpot**.
    * Creates and assigns support tickets with appropriate priority levels.
    * Logs all interactions in a master **Google Sheets** audit trail.
* **Human-like Responses**: Drafts and sends professional, structured email replies via **Gmail**.

---

## 🏗️ System Architecture



```mermaid
graph TD
    A[Gmail Trigger] --> B[Sentiment Analysis]
    B --> C{Is it Urgent?}
    
    C -->|Yes| D[HubSpot: High Priority Ticket]
    D --> E[Slack: Sales Team Alert]
    E --> J[(Google Sheets: Audit Log)]
    
    C -->|No| G[HubSpot: Medium Priority Ticket]
    G --> F
    
    F --> H[(Supabase Vector DB)]
    F --> I[Gmail: Send Automated Reply]
    I --> J[(Google Sheets: Audit Log)]

```

## 🛠️ Tech Stack

* **Automation**: [n8n.io](https://n8n.io/)
* **AI Engine**: OpenAI (GPT-5-mini)
* **Memory/Knowledge**: Supabase (Vector Store) 
* **CRM**: HubSpot
* **Communication**: Gmail & Slack
* **Logging**: Google Sheets

---

## 📋 Workflow Breakdown

### 1. Ingestion & Analysis
The workflow monitors Gmail every minute. Once an email arrives, it is cleaned and sent to the **Sentiment Analysis** node. The AI classifies the intent (Refund, Technical, etc.) and checks for "Urgent" keywords or angry sentiment.

### 2. Intelligent Routing
* **Urgent Path**: Creates a "High Priority" ticket in HubSpot, alerts the team on Slack, and creates a log entry.
* **Standard Path**: Creates a "Medium Priority" ticket and moves directly to the AI Agent for resolution.

### 3. Retrieval-Augmented Generation (RAG)
The AI Agent is equipped with a **Vector Database** tool. For product questions or refund instructions, the agent "searches" the company's actual documentation (stored in Supabase) to ensure the reply is accurate and not hallucinated.

### 4. Automated Response & Logging
The Agent uses the **Gmail Tool** to reply directly to the original thread. Finally, the entire interaction—including the AI's response and the HubSpot ticket number—is logged into a **Google Sheets** CRM log for auditing.

---
## ⚠️ Error Handling

- Implemented safe JSON parsing to prevent workflow failures
- Configured retry on failure for external API calls
- Handles missing or invalid AI responses gracefully

  ---

## ⚙️ Setup Instructions

* **Import**: Import the `Customer_Support_AI_Automation.json` into your n8n instance.

* **API Credentials**:
    * **OpenAI**: For sentiment analysis and the chat agent.
    * **Supabase**: To connect your vector database for RAG.
    * **HubSpot**: To manage tickets and contacts.
    * **Google**: For Gmail (Trigger/Reply) and Sheets (Logging).

* **Vector DB**: Ensure your Supabase `documents` table is populated with your help center articles or FAQs.

* **CRM Fields**: Map your HubSpot Ticket Pipeline IDs and Owner IDs in the HubSpot nodes.

## 📸 Visualizing the Output

### 1. Workflow Execution


<img width="1638" height="813" alt="image" src="https://github.com/user-attachments/assets/5aa8312b-9063-48fb-9280-eaa467b9730b" />

### 2.Email

<img width="1365" height="657" alt="image" src="https://github.com/user-attachments/assets/34487845-0574-47d5-8d9a-6f99367768d3" />

### 3. CRM Data (Hubspot Get Contact)

<img width="1823" height="770" alt="image" src="https://github.com/user-attachments/assets/040728fc-4772-4e1b-8d0e-4d0b31866a8d" />

### 4. Create Ticket (Hubspot)

<img width="1800" height="752" alt="image" src="https://github.com/user-attachments/assets/fcda3065-3a35-4e23-a140-ff63053837bd" />

### 5.Ticket Details

<img width="1813" height="833" alt="image" src="https://github.com/user-attachments/assets/8b844d62-81fa-4cbe-bad1-7d1d48a70c72" />

### 6. Log Entry

<img width="1887" height="607" alt="image" src="https://github.com/user-attachments/assets/bdf21ea6-3b95-41cb-a53e-9917ea432a66" />


### 7. Slack Notifications 

<img width="1362" height="742" alt="image" src="https://github.com/user-attachments/assets/4149cf53-ada7-4499-979b-62f7f0395bba" />

### 8. Urgent :False (Email)

<img width="1386" height="562" alt="image" src="https://github.com/user-attachments/assets/20056623-b685-455d-939d-45d08dfcd5ec" />

### 9. AI Reply to the Issue

<img width="1800" height="662" alt="image" src="https://github.com/user-attachments/assets/e2c30321-0b8b-4645-9f53-a4e7b83140ba" />















