# Install promptfoo on Azure

## Create resource group

az group create --name promptfoo-rg --location germanywestcentral

## Create storage account

STORAGE_ACCOUNT_NAME="promptfoostoragereq"
az storage account create \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group promptfoo-rg \
  --location germanywestcentral \
  --sku Standard_LRS \
  --kind StorageV2

## Create file share

az storage share create \
  --name promptfoo-data \
  --account-name $STORAGE_ACCOUNT_NAME

## Get storage account key

STORAGE_KEY=$(az storage account keys list \
  --resource-group promptfoo-rg \
  --account-name $STORAGE_ACCOUNT_NAME \
  --query "[0].value" -o tsv)

## Create container app environment

az containerapp env create \
  --resource-group promptfoo-rg \
  --name promptfoo-env \
  --location germanywestcentral

## Add storage to the container app environment

az containerapp env storage set \
  --name promptfoo-env \
  --resource-group promptfoo-rg \
  --storage-name promptfoo-storage \
  --azure-file-account-name $STORAGE_ACCOUNT_NAME \
  --azure-file-account-key $STORAGE_KEY \
  --azure-file-share-name promptfoo-data \
  --access-mode ReadWrite

## Create the container app

az containerapp create \
  --resource-group promptfoo-rg \
  --name promptfoo-app \
  --environment promptfoo-env \
  --image ghcr.io/promptfoo/promptfoo:latest \
  --target-port 3000 \
  --ingress external \
  --cpu 2.0 \
  --memory 4Gi \
  --min-replicas 1 \
  --max-replicas 1 \
  --env-vars HOST=0.0.0.0 OPENAI_API_KEY=your-actual-api-key-here

## After that add volume in azure portal

Navigate to the Azure portal, find your container app, go to Application->Volumes->+Add

- Volume type: Azure File

- Name: promptfoostorage

- File share name: promptfoo-storage

## And mount it to the container app

Navigate to the Azure portal, find your container app, go to Application->Containers->Volume mounts and add a volume mount with the following settings

- Volume Name: promptfoo-storage

- Mount Path: /home/promptfoo/.poromptfoo

## Eval error

Somtime I got some eval database error. For this I installed promptfoo with docker locally as documented in promptfoo hosting doc and then copied the local .db file to Azure File share. After that it worked.
