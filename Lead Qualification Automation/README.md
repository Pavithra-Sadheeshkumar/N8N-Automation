# Lead Qualification & Enrichment Automation

An end-to-end **n8n workflow** designed to automate the lead intake, validation, enrichment, and routing process. This system filters out low-quality leads, calculates lead scores, and triggers personalized engagement via Slack, Gmail, and Google Calendar.

---

## 🚀 Overview

This workflow acts as an automated **Sales Development Representative (SDR)**. It handles everything from initial form submission to booking meetings for high-value leads, ensuring your sales team only focuses on qualified opportunities.

---

## ✨ Features

* **Multi-Stage Validation**: Combines internal logic with **Hunter.io** to verify email deliverability and filter out temporary or fake domains.
* **Deep Lead Enrichment**: Uses the Hunter API to retrieve company details, social profiles, and industry group data.
* **CRM Integration**: Syncs with **Airtable** to check existing customer status (New, General, or VIP) and revenue history.
* **Dynamic Lead Scoring**: A custom logic engine categorizes leads into High, Medium, and Low value based on geography, industry (e.g., Software), and email scores.
* **Automated Document Generation**: Creates a personalized "Lead Brief" in Google Docs, converts it to PDF, and stores it in Google Drive.
* **Sales Orchestration**:
    * **High Value**: Immediate Slack alert to Account Executives and automated Google Calendar scheduling.
    * **Medium Value**: Triggers an approval workflow via Gmail for manual manager review.
    * **Low Value**: Automatically added to a "Nurture List" in Google Sheets.

---

## 🛠️ Tech Stack

* **Automation**: [n8n.io](https://n8n.io/)
* **Intelligence**: Hunter.io (Email Verifier & Information Discovery)
* **CRM/Database**: Airtable, Google Sheets
* **Communication**: Slack, Gmail
* **Productivity**: Google Docs, Google Drive, Google Calendar

---

## 📋 Workflow Logic

1.  **Trigger**: Form submission (Name, Email, Company, Interest Area).
2.  **Validation**: Basic regex and domain checks, followed by Hunter.io's 3rd-party verification.
3.  **Enrichment**: Data retrieval via Hunter Combined API and Airtable lookup.
4.  **Classification**:
    * **Switch Logic**: Segments leads by customer type and revenue.
    * **Scoring**: Incremental scoring based on country (US, UK, IN, DE) and industry group (Software/Marketing).
5.  **Action Paths**:
    * **High-Value**: Slack Notification ➡️ Google Calendar Event ➡️ Welcome Email ➡️ 4-day Wait ➡️ Follow-up Email.
    * **Medium-Value**: Manager Approval Request ➡️ If approved, routes to High-Value path.
    * **Low-Value/Discarded**: Logged in Google Sheets for future marketing nurture.
  
```mermaid
graph TD
    %% Trigger
    A[Form Submission] -->|Webhook| B(n8n Workflow Engine)

    %% Validation & Enrichment
    B --> C{Email Validation}
    C -->|Valid| D[Hunter.io API]
    C -->|Invalid| E[(Log: Discarded)]
    
    D -->|Enrich Data| F[Airtable CRM Lookup]
    
    %% Scoring
    F --> G{Lead Scoring Logic}
    G -->|High Value| H[High-Value Path]
    G -->|Medium Value| I[Medium-Value Path]
    G -->|Low Value| J[Low-Value Path]

    %% Actions: High Value
    H --> H1[Slack Alert]
    H --> H2[Google Calendar Invite]
    H --> H3[Gmail: Welcome Series]

    %% Actions: Medium Value
    I --> I1[Gmail: Manager Approval]
    I1 -->|Approved| H
    I1 -->|Denied| E

    %% Actions: Low Value
    J --> J1[(Google Sheets: Nurture List)]

    %% Document Generation (Parallel for High/Med)
    H & I1 --> K[Google Docs Template]
    K --> L[Convert to PDF]
    L --> M[Google Drive Upload]

    %% Final Logging
    H1 & J1 & M --> N[(Master Log: Google Sheets)]

```

    
## ⚙️ Setup

1.  **Import Workflow**: Download the `Lead_Automation_Workflow.json` and import it into your n8n instance.
2.  **Credentials**: Set up API credentials for:
    * Hunter.io
    * Google Cloud (Gmail, Drive, Docs, Calendar, Sheets)
    * Slack (OAuth)
    * Airtable

3.	**Environment Variables**: Ensure the Folder IDs and Spreadsheet IDs in the node parameters match your local setup.

## 📸 Visualizing the Output

### 1. Workflow Execution

<img width="1668" height="702" alt="image" src="https://github.com/user-attachments/assets/eaa094f8-9d7c-4d22-b53f-6087c92338ef" />


### 2.Lead Submission Form

<img width="1207" height="682" alt="image" src="https://github.com/user-attachments/assets/c1d8909a-0cd5-4f28-965b-df9a60a611d5" />

### 3.Enrich Lead Information

<img width="1847" height="782" alt="image" src="https://github.com/user-attachments/assets/a2c8edf4-bb31-4f34-87c7-9e580a63a551" />

### 4. Check for new or existing customer in CRM based on the email address.
       When the email adress is found it returns the data with the details available on CRM otherwise no output is returned and treated as **NEW CUSTOMER**

<img width="1285" height="811" alt="image" src="https://github.com/user-attachments/assets/8d4df975-7ab3-43f8-9a5f-a3b5f361b9a0" />

<img width="1835" height="767" alt="image" src="https://github.com/user-attachments/assets/12a53b60-7503-496b-9c4c-bf697e3a7617" />

### 5.The Lead is classified as NEW,GENERAL OR VIP


<img width="1833" height="815" alt="image" src="https://github.com/user-attachments/assets/287b109e-ebfb-4a29-b837-7e4330c50bc5" />

### 6.Lead score is calculated based on email score,Industry and REgion

<img width="1856" height="720" alt="image" src="https://github.com/user-attachments/assets/4a6b0179-2673-4335-8dcb-c69c92af8a3c" />

### 7.Lead Qualified Brief have been created ion Google Drive and the value has been updated dynamically

<img width="1460" height="466" alt="image" src="https://github.com/user-attachments/assets/9d9e8f97-dfcc-421f-bf39-1a14ebb5c85e" />

### 8. Sample values have been updated
<img width="918" height="813" alt="image" src="https://github.com/user-attachments/assets/bae866c0-1d23-4451-a836-600b9108fb4e" />

### 9.Content Coordinator have been notified

<img width="1146" height="447" alt="image" src="https://github.com/user-attachments/assets/ca80be2c-9ee5-4759-be04-7ac287a29b72" />

### 10.At the same time Account executive have been notified and calendar event have been created and welcome email have been sent to the lead

<img width="1212" height="668" alt="image" src="https://github.com/user-attachments/assets/bb7f19c8-6e01-492a-ac1f-f5ea78f9b275" />

<img width="1852" height="890" alt="image" src="https://github.com/user-attachments/assets/e6f5013a-b316-4fa9-a9ff-aa5f729c38d9" />

<img width="720" height="627" alt="image" src="https://github.com/user-attachments/assets/f02eb02e-4f54-428d-9c3d-fe588e6c29ee" />














