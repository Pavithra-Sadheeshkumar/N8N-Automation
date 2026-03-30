# Order Status Automation System 

This is an automated order management workflow built in **n8n**. The system synchronizes order data from Google Sheets and triggers multi-channel notifications (Gmail, Slack, Asana) based on the real-time status of an order.

## 🚀 Overview
The workflow acts as a central "brain" for e-commerce operations. It fetches order details and routes them through a complex switching logic to handle everything from payment reminders to inventory updates and customer feedback.

---

## ✨ Key Features
* **Payment Reminders:** Automatically sends Gmail reminders for "Pending" orders. If an order remains unpaid for >24 hours, it cancels the order and notifies both the customer and the internal team.
* **Warehouse & Logistics Sync:**
    * Notifies the warehouse team via **Slack** for new "Processing" orders.
    * Automatically creates tasks in **Asana** for order packing.
    * Notifies the logistics team when status changes to "Ready for Pickup."
* **Shipping & Delivery:**
    * Sends tracking URLs to customers once items are marked as "Shipped."
    * Sends a delivery confirmation email once the package is received.
* **Inventory Management:** Automatically calculates and updates stock levels in the Google Sheets inventory database when orders are "Refunded" or "Cancelled."
* **Customer Experience:** Includes a "Wait" mechanism to send an automated feedback survey 24 hours after successful delivery.
---
## 🛠️ Tech Stack
* **n8n:** Workflow Automation.
* **Google Sheets:** Primary database for Order, Payment, and Inventory data.
* **Gmail:** Automated customer communication.
* **Slack:** Internal team alerts (Finance, Logistics, Warehouse).
* **Asana:** Task management for warehouse operations.

---

## 📋 Workflow Logic


### 1. Data Retrieval
The workflow is triggered manually (or can be set via a Cron schedule) and pulls all relevant rows from the **"Order Details"** Google Sheet to begin processing.

### 2. The Switch Engine
Orders are automatically categorized into seven distinct operational paths:

* **Pending:** Triggers a 2-stage reminder system. If payment isn't received, it executes auto-cancellation logic.
* **Processing:** Alerts the Warehouse via **Slack** and creates a dedicated **Asana** task for fulfillment.
* **Shipped:** Automatically emails the DHL tracking number and URL to the customer.
* **Delivered:** Triggers a "Wait" node (24 hours) followed by an automated Feedback request.
* **Refunded/Cancelled:** Triggers an inventory restock calculation and notifies the **Finance** team via Slack.
* **Ready for Pickup:** Coordinates directly with the **Logistics** team for DHL pickup arrangements.

### 3. Safety Checks
To prevent "notification fatigue," each path includes **"If" nodes** (e.g., *Checking notification*). These verify if an alert has already been sent, ensuring customers and teams never receive duplicate messages for the same status.


```mermaid
graph TD
    %% Start and Data Fetch
    Start([Manual Trigger]) --> GetRows[Google Sheets: Get Order Rows]
    GetRows --> MainSwitch{Switch: order_status}

    %% PENDING PATH (The State Machine)
    MainSwitch -- Pending --> PendingSwitch{Switch: Notification Sent}
    
    PendingSwitch -- "Empty/Blank" --> Email1[Gmail: Send 1st Reminder]
    Email1 --> UpdateSent1[Google Sheets: Set 'Sent1']
    UpdateSent1 --> SlackFinance[Slack: Notify Finance Team]

    PendingSwitch -- "Sent1" --> TimeCheck20{Time > 20h?}
    TimeCheck20 -- Yes --> Email2[Gmail: Send 2nd Reminder]
    Email2 --> UpdateSent2[Google Sheets: Set 'Sent2']
    
    PendingSwitch -- "Sent2" --> TimeCheck24{Time > 24h?}
    TimeCheck24 -- Yes --> CancelOrder[Google Sheets: Set Status 'Cancelled']
    CancelOrder --> UpdateSentNo[Google Sheets: Set 'No']
    UpdateSentNo --> LoopCancel[Loop: Inventory Restock Path]

    %% PROCESSING PATH
    MainSwitch -- Processing --> CheckProc{If: Notification != 'Yes'}
    CheckProc -- Yes --> SlackWh[Slack: Notify Warehouse]
    SlackWh --> Asana[Asana: Create Packing Task]
    Asana --> UpdateProcYes[Google Sheets: Set 'Yes']

    %% SHIPPED PATH
    MainSwitch -- Shipped --> CheckShip{If: Notification != 'Yes'}
    CheckShip -- Yes --> GmailShip[Gmail: Send Tracking URL]
    GmailShip --> UpdateShipYes[Google Sheets: Set 'Yes']

    %% DELIVERED PATH
    MainSwitch -- Delivered --> CheckDeliv{If: Notification != 'Yes'}
    CheckDeliv -- Yes --> GmailDeliv[Gmail: Confirm Delivery]
    GmailDeliv --> Wait24[Wait 24 Hours]
    Wait24 --> Feedback[Gmail: Send Feedback Survey]

    %% REFUNDED / CANCELLED (Inventory Logic)
    MainSwitch -- Refunded/Cancelled --> CheckInv{If: Notification != 'Yes'}
    CheckInv -- Yes --> GetProd[Google Sheets: Get Product IDs]
    GetProd --> GetStock[Google Sheets: Get Current Stock]
    GetStock --> CalcStock[Code Node: Match IDs & Calc Total]
    CalcStock --> UpdateInv[Google Sheets: Update Inventory]
    UpdateInv --> UpdateInvYes[Google Sheets: Set 'Yes']
    UpdateInvYes --> SlackFinInv[Slack: Notify Finance - Stock Updated]

    %% READY FOR PICKUP
    MainSwitch -- "Ready for Pickup" --> CheckPick{If: Notification != 'Yes'}
    CheckPick -- Yes --> SlackLog[Slack: Notify Logistics]
    SlackLog --> UpdatePickYes[Google Sheets: Set 'Yes']

    %% Styling
    style Start fill:#f9f,stroke:#333,stroke-width:2px
    style MainSwitch fill:#fff4dd,stroke:#d4a017,stroke-width:2px
    style PendingSwitch fill:#fff4dd,stroke:#d4a017,stroke-width:2px
    style CalcStock fill:#e1f5fe,stroke:#01579b
    style Wait24 fill:#f5f5f5,stroke:#666,stroke-dasharray: 5 5

```
---

## 📸 Visualizing the Output

### 1. Workflow Execution

<img width="1888" height="1007" alt="image" src="https://github.com/user-attachments/assets/66431062-0054-4bf3-9957-2584b7a53524" />


### 2.Routing Conditions

<img width="1242" height="680" alt="image" src="https://github.com/user-attachments/assets/724adc9b-8934-4dfb-9bf1-95fcc0c8a61b" />


### 3.Inventory Data

<img width="527" height="572" alt="image" src="https://github.com/user-attachments/assets/7eeafe56-8ee2-41ce-8420-625426b8265f" />

### 4.Order details(Google Sheets) with Pending Status

<img width="1450" height="152" alt="image" src="https://github.com/user-attachments/assets/274c8e92-021f-447c-89e1-1d3859419ff4" />

### 5.First remainder sent to customer if notification is empty und update to Sent1

<img width="1776" height="672" alt="image" src="https://github.com/user-attachments/assets/ab5e0cb5-7209-4bd3-ae6b-a7ed1e54a118" />

### 6.Alert the Finance Team

<img width="1183" height="595" alt="image" src="https://github.com/user-attachments/assets/69e1927a-b5cb-46f6-9491-92a5d886ead2" />

### 7.If notification is Sent1 :Calculate time difference 

<img width="1112" height="607" alt="image" src="https://github.com/user-attachments/assets/ea511c3c-9aa6-4fbc-88c9-6bb8d889ba45" />

### 8.If Time difference >20 send 2nd Remainder Email and update the status =Sent2

<img width="1788" height="618" alt="image" src="https://github.com/user-attachments/assets/06b520a3-0fe2-40b1-b974-400d84f7d622" />

### 9.Wait 4 hours and get the Payment and Order details from database

<img width="597" height="158" alt="image" src="https://github.com/user-attachments/assets/d3954217-f6bf-46a6-a033-485dee12ea42" />

### 10.Calculate again the time difference .If >24 auto cancel the order

<img width="462" height="543" alt="image" src="https://github.com/user-attachments/assets/b40b86c1-6150-49c0-80f2-363dd82ce0cc" />

### 11.Order details and Inventory Details

<img width="1295" height="178" alt="image" src="https://github.com/user-attachments/assets/5831e47e-46d9-4906-adc4-83905111c57a" />


<img width="642" height="172" alt="image" src="https://github.com/user-attachments/assets/f4cbb5a2-6ecf-4ffb-84f3-615b6333d746" />


<img width="510" height="568" alt="image" src="https://github.com/user-attachments/assets/534cd4b5-6dc5-4886-9933-23723cb5c1c8" />


### 12.Order Details after Auto Cancelling

<img width="1386" height="80" alt="image" src="https://github.com/user-attachments/assets/0f986de1-ae3a-4b32-8c3b-864ff55e94af" />

### 13.Inventory data :Stocks has been updated 

<img width="554" height="566" alt="image" src="https://github.com/user-attachments/assets/07c61d9d-cccc-420a-9d40-9d980c4a8a7f" />

### 14.Inventory updation has been notified in the Order details

<img width="1285" height="107" alt="image" src="https://github.com/user-attachments/assets/2d464b91-fc71-45ef-91d9-e96ddee8701d" />


















## 🔧 Setup Instructions

### 1. Import Workflow
Download the `Order_Status_Automation_Final.json` file from this repository and import it into your n8n instance.

### 2. Configure Credentials
You will need to connect the following accounts within n8n:
* **Google Sheets:** Connect your Google Service account or OAuth2.
* **Gmail:** Set up OAuth2 for the customer service email address.
* **Slack:** Create a Slack App in your workspace and provide the **Bot Token**.
* **Asana:** Connect via Personal Access Token (PAT) or OAuth2.

### 3. Customize IDs
* **Sheet IDs:** Ensure the `documentId` in all Google Sheets nodes is updated to match your specific Spreadsheet ID.
* **Channel IDs:** Update the Slack channel IDs to match your specific workspace channels for Finance, Warehouse, and Logistics.




