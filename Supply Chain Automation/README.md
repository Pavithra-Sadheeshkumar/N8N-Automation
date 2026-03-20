# 🌐 Supply Chain EDI Automation Workflow

An automated supply chain solution built with **n8n** that intercepts EDI (Electronic Data Interchange) messages via webhooks, parses them using **AI**, and synchronizes the data with **Google Sheets** and **Slack**.

---

## 🚀 Overview
This workflow is designed to handle three critical supply chain transactions automatically:

* **Purchase Orders (850/ORDERS):** Logs new orders into the CRM.
* **Return Orders (180/RETANN):** Records return requests and reasons.
* **Order Cancellations (860/ORDCHG):** Updates existing CRM entries to "Cancelled" status and alerts the team via Slack.

* 
