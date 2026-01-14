# HR Command - Azure Deployment Guide

Complete guide for deploying the HR Command backend API to Azure Container Apps with Key Vault secrets management and managed identity.

## Prerequisites

Before starting, ensure you have:
- Azure CLI installed (`az --version`)
- Docker installed (`docker --version`)
- Azure subscription with Contributor access
- Access to create Azure resources

## Initial Setup (One-Time)

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

Add all application secrets to Key Vault:

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

# Azure AD credentials
az keyvault secret set `
  --vault-name $VAULT_NAME `
  --name "AZURE-TENANT-ID" `
  --value "YOUR_AZURE_TENANT_ID"
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
    "PORT=3000"
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

# Grant Key Vault access
az keyvault set-policy `
  --name $VAULT_NAME `
  --object-id $PRINCIPAL_ID `
  --secret-permissions get list
```

### Step 9: Configure Secrets from Key Vault

```powershell
# Get Key Vault URI
$VAULT_URI = az keyvault show --name $VAULT_NAME --query properties.vaultUri -o tsv

# Add secrets from Key Vault
az containerapp secret set `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --secrets `
    "database-url=keyvaultref:${VAULT_URI}secrets/DATABASE-URL,identityref:system" `
    "jwt-secret=keyvaultref:${VAULT_URI}secrets/JWT-SECRET,identityref:system"

# Update container app with environment variables
az containerapp update `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --set-env-vars `
    "DATABASE_URL=secretref:database-url" `
    "JWT_SECRET=secretref:jwt-secret"
```

### Step 10: Build and Deploy Application

```powershell
cd backend

# Build Docker image
docker build -t ${ACR_NAME}.azurecr.io/hr-command-backend:latest .

# Login to ACR
az acr login --name $ACR_NAME

# Push image
docker push ${ACR_NAME}.azurecr.io/hr-command-backend:latest

# Update container app
az containerapp update `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --image ${ACR_NAME}.azurecr.io/hr-command-backend:latest
```

### Step 11: Get Application URL

```powershell
$APP_URL = az containerapp show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query properties.configuration.ingress.fqdn -o tsv

Write-Host "Application deployed at: https://$APP_URL"
Write-Host "Health check: https://$APP_URL/health"
```

## Ongoing Deployments

After initial setup, use GitHub Actions for automatic deployments:

1. Push to `main` branch
2. GitHub Actions builds Docker image
3. Pushes to Azure Container Registry
4. Updates Container App
5. Runs health check

## Monitoring & Troubleshooting

### View Logs

```powershell
az containerapp logs show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --follow
```

### Check Application Status

```powershell
az containerapp show `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP `
  --query properties.runningStatus
```

### Restart Application

```powershell
az containerapp revision restart `
  --name $APP_NAME `
  --resource-group $RESOURCE_GROUP
```

## Security Best Practices

- ✅ Secrets stored in Azure Key Vault
- ✅ Managed Identity (no passwords in code)
- ✅ HTTPS-only ingress
- ✅ Non-root container user
- ✅ SQL firewall rules
- ✅ Regular security updates via GitHub Actions

## Cost Optimization

- Container Apps scale to zero when idle
- Uses consumption-based pricing
- Estimated monthly cost: $20-35 for low traffic applications
