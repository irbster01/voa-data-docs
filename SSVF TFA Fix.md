---
layout: default
title: Service Submission Logger
---

# Service Submission Logger

Chrome extension and Azure Functions system that automatically captures service submission data from internal web applications and stores it in a Fabric Lakehouse.

## Overview

This system consists of two components:

1. **Chrome Extension** - Captures page content when users submit service forms
2. **Azure Function** - Receives captured data and appends it to Fabric Lakehouse files

## Architecture

```
User submits form → Extension captures page text → Azure Function → Lakehouse File API
                                                                    ↓
                                                Files/service_logs/YYYY/MM/DD/service_logs_YYYY-MM-DD.ndjson
```

## Repository Structure

```
ssvf-extension/
├── extension/              # Chrome extension (Vite + React + TypeScript)
│   ├── src/
│   │   ├── config.ts      # API URL and key configuration
│   │   ├── content/       # Content script (form/button detection)
│   │   └── popup/         # Extension popup UI
│   └── manifest.json      # Chrome extension manifest (V3)
│
├── api/                   # Azure Function (Node + TypeScript)
│   ├── CaptureIngest/     # Main function handler
│   ├── shared/            # Lakehouse File API client
│   └── local.settings.json
│
└── test-service-page.html # Local test page
```

## Setup Instructions

### Prerequisites

- Node.js 18+ and npm
- Azure Functions Core Tools (`npm install -g azure-functions-core-tools@4`)
- Azure CLI (run `az login` for authentication)
- Chrome browser

### Extension Setup

```powershell
cd extension
npm install
npm run build
```

**Load the extension in Chrome:**

1. Open Chrome and go to `chrome://extensions`
2. Enable "Developer mode" (toggle in top-right)
3. Click "Load unpacked"
4. Select the `extension/dist` folder

**Configure for your domains:**

Edit `extension/manifest.json` and update the `host_permissions` with your internal web app URLs, then rebuild.

### Azure Function Setup

```powershell
cd api
npm install
```

**Configure environment variables:**

Edit `api/local.settings.json`:

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "API_KEY": "your-secure-api-key-here",
    "ONE_LAKE_URL": "https://onelake.dfs.fabric.microsoft.com/YOUR_WORKSPACE/YOUR_LAKEHOUSE",
    "ONE_LAKE_FILES_BASE": "Files/service_logs"
  }
}
```

**Authentication:**

The function uses `DefaultAzureCredential`, which supports:

- **Azure CLI** (for local dev): Run `az login`
- **Managed Identity** (for Azure deployment): Automatically works when deployed
- **Service Principal**: Set environment variables for CLIENT_ID, TENANT_ID, CLIENT_SECRET

**Build and start:**

```powershell
npm run build
npm start
```

The function will start at `http://localhost:7071`

### Extension Configuration

Update `extension/src/config.ts` with your API details:

```typescript
export const API_URL = 'http://localhost:7071/api/captures';
export const API_KEY = 'your-secure-api-key-here';
```

Rebuild the extension after changes:

```powershell
cd extension
npm run build
```

Then reload the extension in Chrome.

## Testing Guide

### Local End-to-End Test

1. **Start the Azure Function:**
   ```powershell
   cd api
   npm start
   ```

2. **Load the test page:**
   
   Open `test-service-page.html` in Chrome. For the extension to work, temporarily add `file:///*` to the `host_permissions` in `manifest.json`, or host the test page via a local web server.

3. **Verify extension is loaded:**
   
   Click the extension icon in Chrome toolbar. You should see "Service logging extension is active".

4. **Test the capture:**
   
   Fill out the form on the test page and click "Submit Service Record". The form will show a success message.

5. **Verify the data was captured:**
   
   Check Chrome DevTools Console (F12) for `[Service Logger]` messages and Azure Function console for successful append messages.

6. **Verify the file in Fabric Lakehouse:**
   
   Open your Fabric workspace → Lakehouse → Files section → `service_logs/YYYY/MM/DD/` → Open the NDJSON file

## Data Format

### Captured Payload (from extension)

```json
{
  "user_id": "unknown",
  "source_url": "https://example.org/services/submit",
  "captured_at_utc": "2025-12-02T10:30:45.123Z",
  "raw_text": "All visible page text at time of submission..."
}
```

### Stored in Lakehouse (enriched)

```json
{
  "user_id": "unknown",
  "source_url": "https://example.org/services/submit",
  "captured_at_utc": "2025-12-02T10:30:45.123Z",
  "received_at_utc": "2025-12-02T10:30:45.789Z",
  "raw_text": "All visible page text at time of submission..."
}
```

## How It Works

### Extension Content Script

The content script runs on configured pages and:

1. Detects forms and submit buttons on page load
2. Attaches event listeners (without preventing default behavior)
3. On submit, captures `document.body.innerText`
4. Sends data using `navigator.sendBeacon()` or `fetch()` with `keepalive: true`
5. Never blocks or interrupts the actual form submission

### Azure Function

The function:

1. Validates the API key from `x-api-key` header
2. Validates required fields (`source_url`, `raw_text`)
3. Adds `received_at_utc` timestamp
4. Appends the data as NDJSON to the daily log file

### Lakehouse Client

The `lakehouseClient.ts` module:

1. Uses `DefaultAzureCredential()` for authentication
2. Uses `@azure/storage-file-datalake` to interact with the Lakehouse File API
3. Creates the file if it doesn't exist
4. Appends new content using `append()` and `flush()`
5. Organizes files by date: `/Files/service_logs/YYYY/MM/DD/`

## Deployment

### Deploy Azure Function

```powershell
cd api

# Create a function app
az functionapp create \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_FUNCTION_APP_NAME \
  --consumption-plan-location eastus \
  --runtime node \
  --runtime-version 18 \
  --functions-version 4 \
  --os-type Linux

# Enable managed identity
az functionapp identity assign \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_FUNCTION_APP_NAME

# Set application settings
az functionapp config appsettings set \
  --resource-group YOUR_RESOURCE_GROUP \
  --name YOUR_FUNCTION_APP_NAME \
  --settings \
    API_KEY="your-production-api-key" \
    ONE_LAKE_URL="https://onelake.dfs.fabric.microsoft.com/YOUR_WORKSPACE/YOUR_LAKEHOUSE" \
    ONE_LAKE_FILES_BASE="Files/service_logs"

# Deploy
func azure functionapp publish YOUR_FUNCTION_APP_NAME
```

### Update Extension for Production

Update `extension/src/config.ts` with your deployed function URL and rebuild.

## Troubleshooting

### Extension not capturing

- Check Chrome DevTools console for `[Service Logger]` messages
- Verify the page URL matches `manifest.json` permissions
- Ensure the extension is enabled in `chrome://extensions`

### Function not receiving data

- Check CORS settings if calling from non-localhost
- Verify API_KEY matches between extension and function
- Check function logs

### Lakehouse write failures

- Verify you're authenticated: `az account show`
- Check managed identity has proper permissions
- Confirm `ONE_LAKE_URL` format is correct
- Ensure the Lakehouse exists and is accessible

## Next Steps

1. **User identification**: Replace `"user_id": "unknown"` by extracting actual user info from the page
2. **Better parsing**: Extract structured data from forms instead of raw text
3. **Monitoring**: Add Application Insights to the Azure Function
4. **Error handling**: Implement retry logic in the extension
5. **Data processing**: Create Fabric Dataflows or Notebooks to parse and transform the NDJSON files

## License

Internal use only.
