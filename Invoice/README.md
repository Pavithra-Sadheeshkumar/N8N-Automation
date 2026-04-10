# 🧾 AI Invoice Extraction & Billing Automation

This Automation stop manually entering data from invoices. This **n8n workflow** automatically detects new invoice files, uses **Open AI** to read the contents, logs the details into a spreadsheet, and notifies the billing team—all within seconds of a file being uploaded.

---


## ✨ Key Features

* **Automated File Monitoring**: Watches a specific **Google Drive** folder for any new file uploads.
* **Intelligent OCR & Extraction**: Uses an AI-powered **Information Extractor** to pull specific fields like Company Name, Invoice Number, Date, Total Amount, and Payment Method.

* **Database Synchronization**: Automatically appends extracted data as new rows in a **Google Sheets** database.
* **Instant Billing Notifications**: Generates a professional notification email via **Open AI** and sends it to the billing department via **Gmail**.



---

## 🏗️ System Architecture

```mermaid
graph TD
    A[Google Drive: New File] --> B{Check: Is PDF?}
    B -- No --> C[Log Format Error]
    B -- Yes --> D[AI: Extract Data]
    D --> E{Check: Data Valid?}
    E -- No --> F[Log Validation Error & Slack Alert]
    E -- Yes --> G{Check: Duplicate?}
    G -- Yes --> H[Log Duplicate Error]
    G -- No --> I[Google Sheets: Success Log]
    I --> J[AI: Generate Email]
    J --> K[Gmail: Notify Billing]

```

## 🛠️ Tech Stack

* **Automation Engine**: [n8n.io]
* **AI Models**: OpenAI GPT-4o-mini
* **Storage**: Google Drive & Google Sheets
* **Communication**: Gmail

---


## 📋 Workflow Breakdown

### 1. Detection and Extraction
The workflow triggers every minute to check a designated Google Drive folder. When a new invoice is found, it downloads the file and converts the PDF content into readable text.

### 2. AI-Powered Data Mapping
The **Information Extractor** node is configured to find nine specific attributes, including Customer Name, Address, and Line Items. This ensures accuracy even if different vendors use different invoice layouts.

### 3. Record Keeping
Once extracted, the data is formatted and appended to a **Google Sheets** document titled "sample". This creates a reliable, automated paper trail for all incoming expenses.

### 4. Reliability & Data Integrity  

* **File Format Validation**: Only processes PDFs to prevent system crashes and ensure data consistency.
* **Data Completeness**: Automatically verifies that critical fields—specifically the **Invoice No** and **Total Amount**—exist before proceeding to the logging stage.
* **Deduplication**: Queries the existing Google Sheet database to confirm the invoice has not been processed previously, ensuring the same invoice is never paid or logged twice.

### 5. Internal Notification
Finally, the AI drafts a JSON-formatted email subject and body. This is sent to the internal billing contact, providing a summary and a link to the master database for review.

---
## 🔮 Future Improvements

### 1. OCR Support for Scanned Invoices
Integrate OCR capabilities (e.g., OCR.space or Google Vision API) to process scanned PDFs and image-based invoices.

### 2. Expanded Format Support
Extend the system to handle multiple input formats such as:
- PDF (scanned and digital)  
- Images (JPG, PNG)  
- Email attachments

  ---

## 📈 Business Impact

-Reduces manual data entry time by 90%.
-Minimizes human error in financial logging
-Ensures 100% of invoices are captured within 60 seconds of arrival.

---

## ⚙️ Setup Instructions

1. **Import Workflow**: Download the `Invoice.json` file and import it into your n8n canvas.

2. **Credential Configuration**:
   The specific nodes in this workflow require you to link your own credentials. You must set up the following in the n8n **Credentials** menu:
   * **OpenAI API**: Create a new 'OpenAI API' credential using your API Key to power the `Information Extractor` and `Message a model` nodes.
   * **Google Drive & Sheets**: Set up 'Google Sheets OAuth2 API' and 'Google Drive OAuth2 API' to allow the workflow to watch your folders and update your spreadsheets.
   * **Gmail**: Configure 'Gmail OAuth2 API' to enable the system to send the automated billing notifications.
   * **Slack**: Set up 'Slack OAuth2 API' to receive validation error alerts.

3. **Node ID Mapping**:
   After creating your credentials, open each node (Google Drive, OpenAI, Google Sheets, Gmail) and select your newly created credential from the dropdown menu to link them to the specific Node IDs mentioned in the JSON.

4. **Environmental Variables**: 
   If you are self-hosting n8n, ensure your `.env` file is configured with the correct `N8N_ENCRYPTION_KEY` to maintain credential security during the import/export process.
 
## 📸 Visualizing the Output

### 1. Workflow Execution

<img width="1600" height="721" alt="image" src="https://github.com/user-attachments/assets/fd22a550-4ea5-4d8d-a5a8-e9ea264d8b56" />


### 2. Attribute extraction using Open AI

<img width="1828" height="740" alt="image" src="https://github.com/user-attachments/assets/2fb998b5-2f33-4953-b52b-38b1fcb62fc1" />

### 3. Appending the Information to Google sheets

<img width="1879" height="630" alt="image" src="https://github.com/user-attachments/assets/78a86f09-37b9-4b6e-9818-2bf63398b137" />

### 4.Using AI to send notification to billing team

<img width="1482" height="667" alt="image" src="https://github.com/user-attachments/assets/bea869ac-a68c-44ea-98e2-341edc389360" />

---






