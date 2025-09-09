# Azure Cheat Sheet

## Table of Contents
- [Installation and Setup](#installation-and-setup)
- [Azure CLI](#azure-cli)
- [Virtual Machines](#virtual-machines)
- [Storage](#storage)
- [Azure Functions](#azure-functions)
- [Azure Active Directory](#azure-active-directory)
- [ARM Templates](#arm-templates)
- [Best Practices and Security](#best-practices-and-security)
- [Integration with Other Tools](#integration-with-other-tools)
- [Troubleshooting Tips](#troubleshooting-tips)

## Installation and Setup

### Azure CLI Installation
```bash
# Install Azure CLI (Linux)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Install Azure CLI (macOS)
brew install azure-cli

# Install Azure CLI (Windows - PowerShell)
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'

# Install via Docker
docker run -it mcr.microsoft.com/azure-cli

# Verify installation
az --version
```

### Authentication and Configuration
```bash
# Login to Azure
az login

# Login with service principal
az login --service-principal -u <app-id> -p <password> --tenant <tenant>

# List accounts
az account list

# Set default subscription
az account set --subscription "subscription-name-or-id"

# Show current account
az account show

# Set default resource group and location
az configure --defaults group=myResourceGroup location=eastus
```

## Azure CLI

### General Commands
```bash
# Get help
az --help
az vm --help

# List available locations
az account list-locations --output table

# List resource groups
az group list --output table

# Create resource group
az group create --name myResourceGroup --location eastus

# Delete resource group
az group delete --name myResourceGroup --yes --no-wait

# List all resources in subscription
az resource list --output table

# Query results with JMESPath
az vm list --query "[?powerState=='VM running'].name" --output table
```

## Virtual Machines

### VM Management
```bash
# List VMs
az vm list --output table
az vm list --resource-group myResourceGroup

# Create VM
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --size Standard_B1s

# Create Windows VM
az vm create \
    --resource-group myResourceGroup \
    --name myWindowsVM \
    --image Win2019Datacenter \
    --admin-username azureuser \
    --admin-password myPassword123!

# Start/Stop/Restart VM
az vm start --resource-group myResourceGroup --name myVM
az vm stop --resource-group myResourceGroup --name myVM
az vm restart --resource-group myResourceGroup --name myVM

# Deallocate VM (stop billing)
az vm deallocate --resource-group myResourceGroup --name myVM

# Delete VM
az vm delete --resource-group myResourceGroup --name myVM
```

### VM Information
```bash
# Show VM details
az vm show --resource-group myResourceGroup --name myVM

# Get VM sizes available in location
az vm list-sizes --location eastus --output table

# List VM images
az vm image list --output table
az vm image list --publisher Canonical --offer UbuntuServer --sku 18.04-LTS --all

# Get VM IP addresses
az vm list-ip-addresses --resource-group myResourceGroup --name myVM

# Show VM power state
az vm get-instance-view --resource-group myResourceGroup --name myVM --query instanceView.statuses[1]
```

### VM Scale Sets
```bash
# Create VM scale set
az vmss create \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --instance-count 2 \
    --admin-username azureuser \
    --generate-ssh-keys

# Scale VM scale set
az vmss scale --resource-group myResourceGroup --name myScaleSet --new-capacity 5

# List instances in scale set
az vmss list-instances --resource-group myResourceGroup --name myScaleSet --output table
```

## Storage

### Storage Account Management
```bash
# Create storage account
az storage account create \
    --name mystorageaccount \
    --resource-group myResourceGroup \
    --location eastus \
    --sku Standard_LRS

# List storage accounts
az storage account list --output table

# Show storage account
az storage account show --name mystorageaccount --resource-group myResourceGroup

# Get storage account keys
az storage account keys list --account-name mystorageaccount --resource-group myResourceGroup

# Delete storage account
az storage account delete --name mystorageaccount --resource-group myResourceGroup
```

### Blob Storage
```bash
# Create container
az storage container create --name mycontainer --account-name mystorageaccount

# List containers
az storage container list --account-name mystorageaccount --output table

# Upload blob
az storage blob upload \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob \
    --file ~/myfile.txt

# List blobs
az storage blob list --account-name mystorageaccount --container-name mycontainer --output table

# Download blob
az storage blob download \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob \
    --file ~/downloaded-file.txt

# Delete blob
az storage blob delete --account-name mystorageaccount --container-name mycontainer --name myblob

# Generate SAS token
az storage blob generate-sas \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob \
    --permissions r \
    --expiry 2024-12-31T23:59:00Z
```

### File Storage
```bash
# Create file share
az storage share create --name myshare --account-name mystorageaccount

# Upload file
az storage file upload \
    --share-name myshare \
    --source ~/myfile.txt \
    --account-name mystorageaccount

# List files
az storage file list --share-name myshare --account-name mystorageaccount --output table

# Download file
az storage file download \
    --share-name myshare \
    --path myfile.txt \
    --dest ~/downloaded-file.txt \
    --account-name mystorageaccount
```

## Azure Functions

### Function App Management
```bash
# Create function app
az functionapp create \
    --resource-group myResourceGroup \
    --consumption-plan-location eastus \
    --runtime python \
    --runtime-version 3.8 \
    --functions-version 3 \
    --name myfunctionapp \
    --storage-account mystorageaccount

# List function apps
az functionapp list --output table

# Show function app
az functionapp show --name myfunctionapp --resource-group myResourceGroup

# Get function app settings
az functionapp config appsettings list --name myfunctionapp --resource-group myResourceGroup

# Set application setting
az functionapp config appsettings set \
    --name myfunctionapp \
    --resource-group myResourceGroup \
    --settings "SETTING_NAME=value"

# Delete function app
az functionapp delete --name myfunctionapp --resource-group myResourceGroup
```

### Function Deployment
```bash
# Deploy from local Git
az functionapp deployment source config-local-git \
    --name myfunctionapp \
    --resource-group myResourceGroup

# Deploy from GitHub
az functionapp deployment source config \
    --name myfunctionapp \
    --resource-group myResourceGroup \
    --repo-url https://github.com/user/repo \
    --branch master \
    --manual-integration

# Deploy ZIP file
az functionapp deployment source config-zip \
    --name myfunctionapp \
    --resource-group myResourceGroup \
    --src function.zip
```

## Azure Active Directory

### User Management
```bash
# List users
az ad user list --output table

# Create user
az ad user create \
    --display-name "John Doe" \
    --password myPassword123! \
    --user-principal-name johndoe@contoso.onmicrosoft.com

# Show user
az ad user show --id johndoe@contoso.onmicrosoft.com

# Update user
az ad user update \
    --id johndoe@contoso.onmicrosoft.com \
    --display-name "John Smith"

# Delete user
az ad user delete --id johndoe@contoso.onmicrosoft.com
```

### Group Management
```bash
# List groups
az ad group list --output table

# Create group
az ad group create --display-name "MyGroup" --mail-nickname mygroup

# Add member to group
az ad group member add --group MyGroup --member-id user-object-id

# List group members
az ad group member list --group MyGroup --output table

# Remove member from group
az ad group member remove --group MyGroup --member-id user-object-id
```

### Service Principal Management
```bash
# Create service principal
az ad sp create-for-rbac --name myServicePrincipal

# List service principals
az ad sp list --display-name myServicePrincipal

# Show service principal
az ad sp show --id app-id

# Delete service principal
az ad sp delete --id app-id

# Reset service principal credentials
az ad sp credential reset --name myServicePrincipal
```

## ARM Templates

### Template Deployment
```bash
# Deploy template
az deployment group create \
    --resource-group myResourceGroup \
    --template-file template.json \
    --parameters @parameters.json

# Deploy template from URL
az deployment group create \
    --resource-group myResourceGroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.storage/storage-account-create/azuredeploy.json

# Validate template
az deployment group validate \
    --resource-group myResourceGroup \
    --template-file template.json \
    --parameters @parameters.json

# List deployments
az deployment group list --resource-group myResourceGroup --output table

# Show deployment
az deployment group show \
    --resource-group myResourceGroup \
    --name deploymentName
```

### Basic ARM Template Example
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "type": "string",
            "metadata": {
                "description": "Storage Account Name"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2021-04-01",
            "name": "[parameters('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2",
            "properties": {}
        }
    ],
    "outputs": {
        "storageAccountId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
        }
    }
}
```

### Bicep Templates
```bicep
param storageAccountName string
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-04-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
```

## Best Practices and Security

### Security Best Practices
```bash
# Enable system-assigned managed identity
az vm identity assign --resource-group myResourceGroup --name myVM

# Assign role to managed identity
az role assignment create \
    --assignee-object-id managed-identity-object-id \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/subscription-id/resourceGroups/myResourceGroup

# Create Key Vault
az keyvault create \
    --name myKeyVault \
    --resource-group myResourceGroup \
    --location eastus

# Add secret to Key Vault
az keyvault secret set --vault-name myKeyVault --name mySecret --value mySecretValue

# Get secret from Key Vault
az keyvault secret show --vault-name myKeyVault --name mySecret --query value -o tsv
```

### Network Security
```bash
# Create Network Security Group
az network nsg create --resource-group myResourceGroup --name myNSG

# Create NSG rule
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNSG \
    --name myNSGRule \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 80 \
    --access allow

# Associate NSG with subnet
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --name mySubnet \
    --network-security-group myNSG
```

## Integration with Other Tools

### Terraform Integration
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}
```

### Ansible Integration
```yaml
- name: Create Azure VM
  azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM
    vm_size: Standard_B1s
    admin_username: azureuser
    ssh_password_enabled: false
    ssh_public_keys:
      - path: /home/azureuser/.ssh/authorized_keys
        key_data: "ssh-rsa AAAAB3NzaC1yc2E..."
    image:
      offer: UbuntuServer
      publisher: Canonical
      sku: '18.04-LTS'
      version: latest
```

### PowerShell Integration
```powershell
# Install Azure PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser

# Connect to Azure
Connect-AzAccount

# Create resource group
New-AzResourceGroup -Name "myResourceGroup" -Location "East US"

# Create VM
New-AzVm -ResourceGroupName "myResourceGroup" -Name "myVM" -Location "East US" -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -OpenPorts 80,3389
```

## Troubleshooting Tips

### Common Issues and Solutions
```bash
# Check Azure service health
az rest --method get --uri https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ResourceHealth/availabilityStatuses?api-version=2020-05-01

# Get activity logs
az monitor activity-log list --resource-group myResourceGroup

# Check resource limits
az vm list-usage --location eastus --output table

# Debug ARM template deployment
az deployment group show \
    --resource-group myResourceGroup \
    --name myDeployment \
    --query properties.error

# Test network connectivity
az network watcher test-connectivity \
    --source-resource myVM \
    --dest-address 8.8.8.8 \
    --dest-port 80

# Check VM boot diagnostics
az vm boot-diagnostics get-boot-log \
    --resource-group myResourceGroup \
    --name myVM
```

### Monitoring and Logging
```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
    --resource /subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
    --name myDiagnosticSetting \
    --storage-account mystorageaccount \
    --logs '[{"category": "Administrative", "enabled": true}]' \
    --metrics '[{"category": "AllMetrics", "enabled": true}]'

# Query logs with KQL
az monitor log-analytics query \
    --workspace workspace-id \
    --analytics-query 'AzureActivity | where TimeGenerated > ago(1h) | limit 10'
```

## Common Use Cases and Patterns

### Load Balancer Setup
```bash
# Create public IP
az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --allocation-method Static

# Create load balancer
az network lb create \
    --resource-group myResourceGroup \
    --name myLoadBalancer \
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool

# Create health probe
az network lb probe create \
    --resource-group myResourceGroup \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80

# Create load balancer rule
az network lb rule create \
    --resource-group myResourceGroup \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe
```

### Auto-scaling Configuration
```bash
# Create auto-scale setting
az monitor autoscale create \
    --resource-group myResourceGroup \
    --resource /subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet \
    --name myAutoscaleSetting \
    --min-count 2 \
    --max-count 10 \
    --count 3

# Create scale-out rule
az monitor autoscale rule create \
    --resource-group myResourceGroup \
    --autoscale-name myAutoscaleSetting \
    --condition "Percentage CPU > 70 avg 5m" \
    --scale out 1
```

## Official Documentation Links

- [Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/)
- [Azure Virtual Machines Documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/)
- [Azure Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/)
- [Azure Functions Documentation](https://docs.microsoft.com/en-us/azure/azure-functions/)
- [Azure Active Directory Documentation](https://docs.microsoft.com/en-us/azure/active-directory/)
- [Azure Resource Manager Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Azure Security Documentation](https://docs.microsoft.com/en-us/azure/security/)