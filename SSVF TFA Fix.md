# Service Submission Logger

A Chrome extension + Azure Functions backend system that automatically captures service submission data from internal web applications and stores it in a Fabric Lakehouse.

## Overview

This system consists of two main components:

1. **Chrome Extension** (`/extension`) - Captures page content when users submit service forms
2. **Azure Function** (`/api`) - Receives captured data and appends it to Fabric Lakehouse files

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
│   │   ├── content/
│   │   │   └── main.ts    # Content script (form/button detection)
│   │   └── popup/
│   │       ├── index.html
│   │       ├── main.tsx
│   │       └── PopupApp.tsx
│   ├── manifest.json      # Chrome extension manifest (V3)
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── api/                   # Azure Function (Node + TypeScript)
│   ├── CaptureIngest/
│   │   ├── function.json
│   │   └── index.ts       # Main function handler
│   ├── shared/
│   │   ├── lakehouseClient.ts  # Lakehouse File API client
│   │   └── types.ts
│   ├── package.json
│   ├── tsconfig.json
│   ├── host.json
│   └── local.settings.json
│
└── test-service-page.html # Local test page
```

## Setup Instructions

### Prerequisites

- Node.js 18+ and npm
- Azure Functions Core Tools (`npm install -g azure-functions-core-tools@4`)
- Azure CLI (`az login` for authentication)
- Chrome browser

### 1. Extension Setup

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

Edit `extension/manifest.json` and update the `host_permissions` and `content_scripts.matches` arrays with your actual internal web app URLs:

```json
"host_permissions": [
  "https://your-lsndc-domain.com/*",
  "https://your-servicepoint-domain.com/*"
]
```

Then rebuild: `npm run build` and reload the extension in Chrome.

### 2. Azure Function Setup

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

Replace:
- `YOUR_WORKSPACE` with your Fabric workspace name
- `YOUR_LAKEHOUSE` with your Lakehouse name
- `your-secure-api-key-here` with a strong API key

**Authentication:**

The function uses `DefaultAzureCredential`, which supports:

1. **Azure CLI** (for local dev): Run `az login`
2. **Managed Identity** (for Azure deployment): Automatically works when deployed
3. **Service Principal**: Set environment variables:
   ```
   AZURE_CLIENT_ID
   AZURE_TENANT_ID
   AZURE_CLIENT_SECRET
   ```

**Build and start:**

```powershell
npm run build
npm start
```

The function will start at `http://localhost:7071`

### 3. Extension Configuration

Update `extension/src/config.ts` with your API details:

```typescript
export const API_URL = 'http://localhost:7071/api/captures';  // or your deployed URL
export const API_KEY = 'your-secure-api-key-here';            // must match function config
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
   
   Wait for: `Functions: CaptureIngest: [POST] http://localhost:7071/api/captures`

2. **Load the test page:**
   
   Open `test-service-page.html` in Chrome (file:// URL is fine for testing)
   
   **Important:** For the extension to work, you need to either:
   
   - Temporarily add `file:///*` to the `host_permissions` in `manifest.json`, OR
   - Host the test page via a local web server that matches your configured domains

3. **Verify extension is loaded:**
   
   - Click the extension icon in Chrome toolbar
   - You should see "Service logging extension is active"

4. **Test the capture:**
   
   - Fill out the form on the test page
   - Click "Submit Service Record"
   - The form will show a success message

5. **Verify the data was captured:**
   
   **In Chrome DevTools (F12):**
   ```
   [Service Logger] Content script loaded on: file:///...
   [Service Logger] Found 1 form(s), attaching submit listeners
   [Service Logger] Form 1 submitted
   [Service Logger] Capturing submission data
   [Service Logger] Data sent via sendBeacon
   ```

   **In Azure Function console:**
   ```
   [2025-12-02T...] Executing 'Functions.CaptureIngest'
   [2025-12-02T...] Received capture: { source_url: '...', captured_at: '...', text_length: 1234 }
   [2025-12-02T...] Appended 1456 bytes to Files/service_logs/2025/12/02/service_logs_2025-12-02.ndjson at offset 0
   [2025-12-02T...] Successfully appended to: Files/service_logs/2025/12/02/service_logs_2025-12-02.ndjson
   [2025-12-02T...] Executed 'Functions.CaptureIngest' (Succeeded)
   ```

6. **Verify the file in Fabric Lakehouse:**
   
   - Open your Fabric workspace
   - Navigate to your Lakehouse
   - Go to **Files** section (not Tables)
   - Browse to `service_logs/2025/12/02/`
   - Open `service_logs_2025-12-02.ndjson`
   - You should see one JSON object per line

### Testing with Real Internal Apps

1. Update `manifest.json` with your actual app URLs
2. Rebuild and reload the extension
3. Navigate to one of your internal apps
4. Open DevTools Console to see extension logs
5. Submit a service form
6. Verify in Function logs and Lakehouse

## Data Format

### Captured Payload (from extension)

```json
{
  "user_id": "unknown",
  "source_url": "https://lsndc.example.org/services/submit",
  "captured_at_utc": "2025-12-02T10:30:45.123Z",
  "raw_text": "All visible page text at time of submission..."
}
```

### Stored in Lakehouse (enriched)

```json
{
  "user_id": "unknown",
  "source_url": "https://lsndc.example.org/services/submit",
  "captured_at_utc": "2025-12-02T10:30:45.123Z",
  "received_at_utc": "2025-12-02T10:30:45.789Z",
  "raw_text": "All visible page text at time of submission..."
}
```

## How It Works

### Extension Content Script

The content script (`extension/src/content/main.ts`) runs on configured pages and:

1. Detects forms and submit buttons on page load
2. Attaches event listeners (without preventing default behavior)
3. On submit, captures `document.body.innerText`
4. Sends data using:
   - First attempt: `navigator.sendBeacon()` (most reliable for page unload)
   - Fallback: `fetch()` with `keepalive: true`
5. Never blocks or interrupts the actual form submission

### Azure Function

The function (`api/CaptureIngest/index.ts`):

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

# Create a function app (one-time setup)
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

# Grant the managed identity access to your Lakehouse
# (Use Azure portal to assign "Storage Blob Data Contributor" role)

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

1. Update `extension/src/config.ts` with your deployed function URL:
   ```typescript
   export const API_URL = 'https://YOUR_FUNCTION_APP_NAME.azurewebsites.net/api/captures';
   export const API_KEY = 'your-production-api-key';
   ```

2. Rebuild: `npm run build`

3. Package for distribution or deploy to Chrome Web Store

## Troubleshooting

### Extension not capturing

- Check Chrome DevTools console for `[Service Logger]` messages
- Verify the page URL matches `manifest.json` permissions
- Ensure the extension is enabled in `chrome://extensions`

### Function not receiving data

- Check CORS settings if calling from non-localhost
- Verify API_KEY matches between extension and function
- Check function logs: `func start` should show incoming requests

### Lakehouse write failures

- Verify you're authenticated: `az account show`
- Check managed identity has proper permissions in Azure
- Confirm `ONE_LAKE_URL` format is correct
- Ensure the Lakehouse exists and is accessible

### File not appearing in Lakehouse

- It can take a few moments for files to appear in Fabric UI
- Check the exact path: `Files/service_logs/YYYY/MM/DD/`
- Verify the function logs show "Successfully appended to: ..."

## Next Steps

1. **User identification**: Replace `"user_id": "unknown"` by extracting actual user info from the page or session
2. **Better parsing**: Instead of `document.body.innerText`, extract structured data from forms
3. **Monitoring**: Add Application Insights to the Azure Function
4. **Error handling**: Implement retry logic in the extension
5. **Data processing**: Create Fabric Dataflows or Notebooks to parse and transform the NDJSON files

## License

Internal use only.
