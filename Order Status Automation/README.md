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

``` Mermaid
    graph TD
    %% Trigger
    Start([Manual Trigger / Schedule]) --> GetRows[Get Row s from Google Sheets]
    GetRows --> Switch{Switch: Order Status}

    %% Path 1: Pending
    Switch -- Pending --> Remainder1[Send 1st Gmail Reminder]
    Remainder1 --> SlackPay[Notify Payment Team - Slack]
    SlackPay --> CalcTime[Calculate Time Difference]
    CalcTime --> TimeCheck{Is it > 24 Hours?}
    TimeCheck -- Yes --> CancelOrder[Update Sheet: Cancelled]
    CancelOrder --> NotifyCustCancel[Notify Customer - Gmail]
    CancelOrder --> NotifyTeamCancel[Notify Team - Slack]

    %% Path 2: Processing
    Switch -- Processing --> NewOrderCheck{Check: Notification Sent?}
    NewOrderCheck -- No --> NotifyWh[Notify Warehouse - Slack]
    NotifyWh --> AsanaTask[Create Asana Packing Task]

    %% Path 3: Shipped
    Switch -- Shipped --> ShipCheck{Check: Notification Sent?}
    ShipCheck -- No --> GmailTrack[Send Tracking URL - Gmail]
    GmailTrack --> UpdateNotif1[Update Sheet: Notification Sent]

    %% Path 4: Delivered
    Switch -- Delivered --> DelivCheck{Check: Notification Sent?}
    DelivCheck -- No --> GmailDeliv[Send Delivery Email]
    GmailDeliv --> Wait24[Wait 24 Hours]
    Wait24 --> Feedback[Send Feedback Form - Gmail]

    %% Path 5: Refunded / Cancelled (Inventory)
    Switch -- Refunded/Cancelled --> AlreadyNotified{Already Notified?}
    AlreadyNotified -- No --> GetProd[Get Product ID & Quantity]
    GetProd --> GetStock[Get Current Inventory Stock]
    GetStock --> CalcStock[Code Node: Calculate New Total]
    CalcStock --> UpdateInv[Update Google Sheets Inventory]
    UpdateInv --> NotifyFin[Notify Finance Team - Slack]

    %% Path 6: Ready for Pickup
    Switch -- Ready for Pickup --> PickupCheck{Check: Notification Sent?}
    PickupCheck -- No --> NotifyLog[Notify Logistics Team - Slack]
    NotifyLog --> UpdateNotif2[Update Sheet: Notification Sent]

    %% Styling
    style Start fill:#f9f,stroke:#333,stroke-width:2px
    style Switch fill:#fff4dd,stroke:#d4a017,stroke-width:2px
    style CalcStock fill:#e1f5fe,stroke:#01579b
```

---

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

