# Lab 4: PhotoPipe — Event-Driven Image Processing Pipeline
**CST8917 — Serverless Applications | Winter 2026**  
**Student:** Sandra Rochel Nyabeng mineme  



## Overview

PhotoPipe is an event-driven image processing pipeline built on Azure. When a user uploads an image through the web application, Azure Event Grid automatically routes the event to two Azure Functions; one that generates image metadata and one that logs the activity to Table Storage for auditing.



## Architecture

```
Browser → Blob Storage (image-uploads)
              ↓ BlobCreated event
         Event Grid System Topic
              ↓                    ↓
    Subscription 1            Subscription 2
    (.jpg/.png only)          (all blob events)
              ↓                    ↓
    process-image fn         audit-log fn
              ↓                    ↓
    Blob Storage             Table Storage
    (image-results)          (processinglog)
```



## Services Used

| Service | Role |
|||
| Blob Storage (image-uploads) | Receives uploaded images from the web client |
| Blob Storage (image-results) | Stores processing metadata JSON files |
| Event Grid System Topic | Captures BlobCreated events from the storage account |
| Event Grid Subscription 1 | Filters .jpg/.png files and triggers image processing |
| Event Grid Subscription 2 | Captures all blob events for audit logging |
| Azure Function (process-image) | Analyzes images and writes metadata to results container |
| Azure Function (audit-log) | Logs all upload events to Table Storage |
| Table Storage (processinglog) | Stores the audit trail of all processed events |
| Web Client (client.html) | PhotoPipe web app for uploading images and viewing results |



## Project Structure

```
PhotoPipeFunctionApp/
├── function_app.py           # Azure Functions (process-image, audit-log, get-results, get-audit-log, health)
├── requirements.txt          # Python dependencies
├── test-function.http        # REST Client test requests
├── client.html               # PhotoPipe web application
├── local.settings.example.json  # Settings template with placeholder values
└── README.md                 # This file
```



## Prerequisites

- Python 3.11 or 3.12
- Azure Functions Core Tools
- VS Code with Azure Functions extension
- An active Azure subscription
- Azurite (for local development)



## Local Setup

**1. Clone the repository**
```bash
git clone <your-repo-url>
cd PhotoPipeFunctionApp
```

**2. Create and activate a virtual environment**
```bash
python -m venv .venv

# Mac/Linux
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Configure local settings**

Copy `local.settings.example.json` to `local.settings.json` and fill in your values:
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "STORAGE_CONNECTION_STRING": "<your-storage-account-connection-string>"
  },
  "Host": {
    "CORS": "*"
  }
}
```

**5. Start Azurite and run locally**

Press `F1` → **Azurite: Start**, then press `F5` to start the function app.



## Azure Deployment

**1. Create a Function App in Azure**

Press `F1` → **Azure Functions: Create Function App in Azure... (Advanced)**

| Setting | Value |
|||
| Runtime | Python 3.12 |
| OS | Linux |
| Hosting plan | Consumption |
| Region | Canada Central |

**2. Add Application Settings**

In the Azure Portal → Function App → Environment variables → add:

| Name | Value |
|||
| `STORAGE_CONNECTION_STRING` | Your storage account connection string |
| `ENABLE_ORYX_BUILD` | `true` |
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | `true` |

**3. Deploy**

Press `F1` → **Azure Functions: Deploy to Function App**

**4. Verify**

Browse to:
```
https://<your-function-app>.azurewebsites.net/api/health
```
Expected response:
```json
{"status": "healthy", "service": "PhotoPipe Function App"}
```



## Storage Account Setup

| Container | Access Level |
|||
| image-uploads | Blob (anonymous read) |
| image-results | Private |

**CORS configuration (Blob service):**

| Field | Value |
|||
| Allowed origins | * |
| Allowed methods | GET, PUT, OPTIONS, HEAD |
| Allowed headers | * |
| Exposed headers | * |
| Max age | 3600 |



## Event Grid Configuration

**System Topic:** `photopipe-blob-events` (connected to storage account)

**Subscription 1 — process-image-sub:**
- Event type: Blob Created
- Subject begins with: `/blobServices/default/containers/image-uploads`
- Advanced filter: `subject` ends with `.jpg` or `.png`
- Endpoint: `process-image` Azure Function

**Subscription 2 — audit-log-sub:**
- Event type: Blob Created
- Subject begins with: `/blobServices/default/containers/image-uploads`
- No suffix filter (captures all file types)
- Endpoint: `audit-log` Azure Function



## Web Client Setup

1. Open `client.html` in a browser
2. Fill in the configuration section:
   - **Storage Account Name:** your storage account name
   - **SAS Token:** generated from Azure Portal (Shared access signature)
   - **Function App URL:** `https://<your-function-app>.azurewebsites.net`



## Testing

| Test | Expected Result |
|||
| Upload a .jpg | Results tab shows metadata card + Audit Log shows entry |
| Upload a .png | Results tab shows metadata card + Audit Log shows entry |
| Upload a .txt or .pdf | Audit Log shows entry only — no Results card |



## Security Notes

- `local.settings.json` is excluded from version control — never commit it
- SAS tokens are short-lived — regenerate as needed
- In production, restrict CORS origins and use managed identities instead of connection strings



## Demo Video

https://youtu.be/aONu0Euppso



## Dependencies

```
azure-functions
azure-storage-blob
azure-data-tables
```