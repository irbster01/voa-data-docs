# HR Command - Azure Deployment Guide

## 🚀 Initial Setup (One-time)

### Prerequisites
- Azure CLI installed: `az --version`
- Docker installed: `docker --version`
- Azure subscription with permissions to create resources

### Step 1: Login to Azure

```powershell
az login
az account set --subscription "YOUR_SUBSCRIPTION_ID"
```

### Step 2: Create Resource Group

```powershell
$LOCATION = "eastus"
$RESOURCE_GROUP = "hr-command-rg"

az group create --name $RESOURCE_GROUP --location $LOCATION
```

### Step 3: Create Azure Container Registry

```powershell
$ACR_NAME = "hrcommandacr"

az acr create `
  --resource-group $RESOURCE_GROUP `
  --name $ACR_NAME `
  --sku Basic `
  --admin-enabled true
```

### Step 4: Create Azure Key Vault

```powershell
$VAULT_NAME = "hr-command-vault"

az keyvault create `
  --name $VAULT_NAME `
  --resource-group $RESOURCE_GROUP `
  --location $LOCATION
```

### Step 5: Store Secrets in Key Vault

```powershell
# Database connection string
az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "DATABASE-URL" `
  --value "sqlserver://voanla-ops-sql.database.windows.net:1433;database=voanla-ops-db;encrypt=true"

# JWT Secret
az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "JWT-SECRET" `
  --value "YOUR_SECURE_JWT_SECRET_HERE"

# Paycor API credentials
az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "PAYCOR-CLIENT-ID" `
  --value "YOUR_PAYCOR_CLIENT_ID"

az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "PAYCOR-CLIENT-SECRET" `
  --value "YOUR_PAYCOR_CLIENT_SECRET"

az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "PAYCOR-SUBSCRIPTION-KEY" `
  --value "YOUR_PAYCOR_SUBSCRIPTION_KEY"

az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "PAYCOR-REFRESH-TOKEN" `
  --value "YOUR_PAYCOR_REFRESH_TOKEN"

# Azure AD credentials
az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "AZURE-TENANT-ID" `
  --value "YOUR_AZURE_TENANT_ID"

az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "AZURE-CLIENT-ID" `
  --value "YOUR_AZURE_CLIENT_ID"

az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "AZURE-CLIENT-SECRET" `
  --value "YOUR_AZURE_CLIENT_SECRET"
```

### Step 6: Create Container Apps Environment

```powershell
$ENVIRONMENT_NAME = "hr-command-env"

az containerapp env create `
  --name $ENVIRONMENT_NAME `
  --resource-group $RESOURCE_GROUP `
  --location $LOCATION
```

### Step 7: Create Container App

```powershell
$APP_NAME = "hr-command-api"

# Get ACR credentials
$ACR_USERNAME = az acr credential show --name $ACR_NAME --query username -o tsv
$ACR_PASSWORD = az acr credential show --name $ACR_NAME --query passwords[0].value -o tsv

# Create the container app
az containerapp create `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --environment $ENVIRONMENT_NAME `
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest `
  --target-port 3000 `
  --ingress external `
  --registry-server "$ACR_NAME.azurecr.io" `
  --registry-username $ACR_USERNAME `
  --registry-password $ACR_PASSWORD `
  --cpu 0.5 `
  --memory 1.0Gi `
  --min-replicas 1 `
  --max-replicas 10 `
  --env-vars `
    "NODE_ENV=production" `
    "PORT=3000" `
    "PAYCOR_BASE_URL=https://apis.paycor.com" `
    "PAYCOR_TOKEN_URL=https://apis.paycor.com/sts/v1/common/token" `
    "PAYCOR_LEGAL_ENTITY_ID=141164"
```

### Step 8: Enable Managed Identity

```powershell
# Enable system-assigned managed identity
az containerapp identity assign `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --system-assigned

# Get the identity principal ID
$PRINCIPAL_ID = az containerapp identity show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query principalId -o tsv

# Grant Key Vault access to the managed identity
az keyvault set-policy `
  --name $VAULT_NAME `
  --object-id $PRINCIPAL_ID `
  --secret-permissions get list
```

### Step 9: Configure Secrets from Key Vault

```powershell
# Get Key Vault URI
$VAULT_URI = az keyvault show --name $VAULT_NAME --query properties.vaultUri -o tsv

# Update container app with Key Vault references
az containerapp update `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --set-env-vars `
    "DATABASE_URL=secretref:database-url" `
    "JWT_SECRET=secretref:jwt-secret" `
    "PAYCOR_CLIENT_ID=secretref:paycor-client-id" `
    "PAYCOR_CLIENT_SECRET=secretref:paycor-client-secret" `
    "PAYCOR_SUBSCRIPTION_KEY=secretref:paycor-subscription-key" `
    "PAYCOR_REFRESH_TOKEN=secretref:paycor-refresh-token" `
    "AZURE_TENANT_ID=secretref:azure-tenant-id" `
    "AZURE_CLIENT_ID=secretref:azure-client-id" `
    "AZURE_CLIENT_SECRET=secretref:azure-client-secret"

# Add secrets from Key Vault
az containerapp secret set `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --secrets `
    "database-url=keyvaultref:${VAULT_URI}secrets/DATABASE-URL,identityref:system" `
    "jwt-secret=keyvaultref:${VAULT_URI}secrets/JWT-SECRET,identityref:system" `
    "paycor-client-id=keyvaultref:${VAULT_URI}secrets/PAYCOR-CLIENT-ID,identityref:system" `
    "paycor-client-secret=keyvaultref:${VAULT_URI}secrets/PAYCOR-CLIENT-SECRET,identityref:system" `
    "paycor-subscription-key=keyvaultref:${VAULT_URI}secrets/PAYCOR-SUBSCRIPTION-KEY,identityref:system" `
    "paycor-refresh-token=keyvaultref:${VAULT_URI}secrets/PAYCOR-REFRESH-TOKEN,identityref:system" `
    "azure-tenant-id=keyvaultref:${VAULT_URI}secrets/AZURE-TENANT-ID,identityref:system" `
    "azure-client-id=keyvaultref:${VAULT_URI}secrets/AZURE-CLIENT-ID,identityref:system" `
    "azure-client-secret=keyvaultref:${VAULT_URI}secrets/AZURE-CLIENT-SECRET,identityref:system"
```

### Step 10: Configure SQL Database Firewall

```powershell
# Get Container App outbound IPs
$OUTBOUND_IPS = az containerapp show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query properties.outboundIpAddresses -o tsv

# Add firewall rules for each IP
foreach ($IP in $OUTBOUND_IPS -split ',') {
  $ruleName = "ContainerApp-$($IP.Replace('.', '-'))"
  az sql server firewall-rule create `
    --resource-group "YOUR_SQL_RESOURCE_GROUP" `
    --server "voanla-ops-sql" `
    --name $ruleName `
    --start-ip-address $IP `
    --end-ip-address $IP
}
```

### Step 11: Build and Push Initial Image

```powershell
cd backend

# Build Docker image
docker build -t ${ACR_NAME}.azurecr.io/hr-command-backend:latest .

# Login to ACR
az acr login --name $ACR_NAME

# Push image
docker push ${ACR_NAME}.azurecr.io/hr-command-backend:latest

# Update container app to use the image
az containerapp update `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --image ${ACR_NAME}.azurecr.io/hr-command-backend:latest
```

### Step 12: Run Database Migrations

```powershell
# Create a temporary container app job to run migrations
az containerapp job create `
  --name "hr-command-migrate" `
  --resource-group $RESOURCE_GROUP `
  --environment $ENVIRONMENT_NAME `
  --trigger-type Manual `
  --replica-timeout 300 `
  --image ${ACR_NAME}.azurecr.io/hr-command-backend:latest `
  --command "npx" "prisma" "migrate" "deploy" `
  --registry-server "$ACR_NAME.azurecr.io" `
  --registry-username $ACR_USERNAME `
  --registry-password $ACR_PASSWORD `
  --secrets `
    "database-url=keyvaultref:${VAULT_URI}secrets/DATABASE-URL,identityref:system" `
  --env-vars "DATABASE_URL=secretref:database-url"

# Run the migration job
az containerapp job start --name "hr-command-migrate" --resource-group $RESOURCE_GROUP
```

### Step 13: Set up GitHub Actions

```powershell
# Create service principal for GitHub Actions
$SP = az ad sp create-for-rbac `
  --name "hr-command-github-actions" `
  --role contributor `
  --scopes "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --sdk-auth

# Copy the JSON output and add it as a GitHub secret named AZURE_CREDENTIALS
Write-Host "Add this JSON as GitHub secret 'AZURE_CREDENTIALS':"
Write-Host $SP
```

### Step 14: Get Application URL

```powershell
$APP_URL = az containerapp show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query properties.configuration.ingress.fqdn -o tsv

Write-Host "✅ Application deployed at: https://$APP_URL"
Write-Host "✅ Health check: https://$APP_URL/health"
Write-Host "✅ API base: https://$APP_URL/api"
```

## 🔄 Ongoing Deployments

After initial setup, deployments are automatic:

1. Push to `master` branch
2. GitHub Actions builds Docker image
3. Pushes to Azure Container Registry
4. Updates Container App
5. Runs smoke test

## 📊 Monitoring & Logs

```powershell
# View logs
az containerapp logs show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --follow

# View metrics
az monitor metrics list `
  --resource "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.App/containerApps/$APP_NAME" `
  --metric-names Requests

# Check app status
az containerapp show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query properties.runningStatus
```

## 🛠️ Troubleshooting

### Check application logs
```powershell
az containerapp logs show --name $APP_NAME --resource-group $RESOURCE_GROUP --tail 100
```

### Check replica status
```powershell
az containerapp replica list --name $APP_NAME --resource-group $RESOURCE_GROUP
```

### Test health endpoint
```powershell
curl https://YOUR_APP_URL/health
```

### Restart application
```powershell
az containerapp revision restart `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP
```

## 💰 Cost Optimization

- Container Apps scale to zero when idle
- Uses consumption-based pricing
- Estimated monthly cost: $20-35 (low traffic)

## 🔐 Security Best Practices

✅ Secrets stored in Azure Key Vault  
✅ Managed Identity (no passwords in code)  
✅ HTTPS-only ingress  
✅ Non-root container user  
✅ SQL firewall rules  
✅ Regular security updates via GitHub Actions

## 📱 Update Chrome Extension

After deployment, update extension API URL:

```javascript
// extension/src/config.ts or similar
const API_BASE_URL = 'https://YOUR_APP_URL/api';
```

Rebuild and reload extension in Chrome.
