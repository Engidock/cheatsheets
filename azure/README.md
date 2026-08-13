# Azure Cheatsheet

Complete, detailed reference guide for Microsoft Azure Cloud Computing — Azure CLI commands, resource management, networking, containers, identity, monitoring, and cost management.

## 🎯 Azure Fundamentals

### Azure Setup & Account

Azure account setup:

1. Create Azure account at portal.azure.com
2. Set up subscription(s)
3. Create resource group(s)
4. Configure billing alerts
5. Enable Azure AD MFA
6. Set up Azure Policy
7. Configure resource tagging
8. Enable Azure Security Center

Azure CLI installation:

```bash
# macOS
brew install azure-cli

# Linux (Ubuntu/Debian)
sudo apt-get install azure-cli

# Windows
# Download installer from https://aka.ms/installazurecliwindows

# Verify installation
az --version

# Login to Azure
az login

# Set default subscription
az account set --subscription "subscription-id"

# View current account
az account show
```

Resource groups:

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# List resource groups
az group list --output table

# Get resource group details
az group show --name myResourceGroup

# Delete resource group
az group delete --name myResourceGroup --yes
```

## 🖥️ Virtual Machines

### VM Management

Create virtual machine:

```bash
# Create Windows VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Win2019Datacenter \
  --admin-username azureuser \
  --admin-password P@ssw0rd1234!

# Create Linux VM (Ubuntu)
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Create with custom image
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image myCustomImage \
  --size Standard_D2s_v3
```

VM operations:

```bash
# List VMs
az vm list --resource-group myResourceGroup --output table

# Get VM details
az vm show --resource-group myResourceGroup --name myVM

# Start VM
az vm start --resource-group myResourceGroup --name myVM

# Stop VM
az vm stop --resource-group myResourceGroup --name myVM

# Deallocate VM (stop & free compute)
az vm deallocate --resource-group myResourceGroup --name myVM

# Restart VM
az vm restart --resource-group myResourceGroup --name myVM

# Delete VM
az vm delete --resource-group myResourceGroup --name myVM --yes
```

SSH & RDP access:

```bash
# SSH into Linux VM
ssh azureuser@public-ip-address

# Get public IP
az vm list-ip-addresses \
  --resource-group myResourceGroup \
  --name myVM \
  --output table

# Open port in NSG (Network Security Group)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name AllowSSH \
  --priority 1000 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 22
```

## 📦 Storage Accounts

### Storage Management

Storage account operations:

```bash
# Create storage account
az storage account create \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --location eastus \
  --sku Standard_LRS

# List storage accounts
az storage account list --output table

# Get storage account key
az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount

# Update storage account
az storage account update \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --minimum-tls-version TLS1_2

# Delete storage account
az storage account delete \
  --resource-group myResourceGroup \
  --name mystorageaccount
```

Blob storage:

```bash
# Create container
az storage container create \
  --account-name mystorageaccount \
  --name mycontainer

# List containers
az storage container list \
  --account-name mystorageaccount \
  --output table

# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt

# Download blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded-file.txt

# List blobs
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --output table

# Delete blob
az storage blob delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt
```

## 🗄️ Azure SQL & Cosmos DB

### Database Services

Azure SQL Database:

```bash
# Create SQL server
az sql server create \
  --resource-group myResourceGroup \
  --name mysqlserver \
  --location eastus \
  --admin-user sqladmin \
  --admin-password P@ssw0rd1234!

# Create database
az sql db create \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name mydb \
  --service-objective S0

# List databases
az sql db list \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --output table

# Get connection string
az sql db show-connection-string \
  --client psql \
  --server mysqlserver \
  --name mydb

# Create firewall rule
az sql server firewall-rule create \
  --resource-group myResourceGroup \
  --server mysqlserver \
  --name AllowMyIP \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255
```

Cosmos DB:

```bash
# Create Cosmos DB account
az cosmosdb create \
  --resource-group myResourceGroup \
  --name mycosmosdb \
  --kind GlobalDocumentDB

# Create database
az cosmosdb database create \
  --resource-group myResourceGroup \
  --name mycosmosdb \
  --db-name mydb

# Create container
az cosmosdb collection create \
  --resource-group myResourceGroup \
  --collection-name mycollection \
  --db-name mydb \
  --name mycosmosdb

# List accounts
az cosmosdb list --output table

# Get connection string
az cosmosdb keys list \
  --resource-group myResourceGroup \
  --name mycosmosdb \
  --type connection-strings
```

## 🌐 Virtual Networks & Networking

### Network Configuration

Virtual network setup:

```bash
# Create virtual network
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.0.0/24

# List virtual networks
az network vnet list --output table

# Create additional subnet
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet2 \
  --address-prefix 10.0.1.0/24

# Get VNet details
az network vnet show \
  --resource-group myResourceGroup \
  --name myVNet
```

Network Security Groups:

```bash
# Create NSG
az network nsg create \
  --resource-group myResourceGroup \
  --name myNetworkSecurityGroup

# Add inbound rule (HTTP)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name AllowHTTP \
  --priority 100 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80

# Add inbound rule (HTTPS)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name AllowHTTPS \
  --priority 101 \
  --destination-port-ranges 443 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp

# List rules
az network nsg rule list \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --output table
```

## ☁️ App Service & Web Apps

### Web Application Hosting

App Service plan:

```bash
# Create App Service plan
az appservice plan create \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --sku B1 \
  --is-linux

# List App Service plans
az appservice plan list --output table

# Scale up App Service plan
az appservice plan update \
  --resource-group myResourceGroup \
  --name myAppServicePlan \
  --sku B2
```

Web app creation:

```bash
# Create web app (.NET)
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --runtime "DOTNET|5.0"

# Create web app (Node.js)
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --runtime "node|14-lts"

# Create web app (Python)
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name myWebApp \
  --runtime "PYTHON|3.9"

# List web apps
az webapp list --output table

# Deploy from local git
az webapp deployment source config-local-git \
  --resource-group myResourceGroup \
  --name myWebApp
```

Web app configuration:

```bash
# Set app settings
az webapp config appsettings set \
  --resource-group myResourceGroup \
  --name myWebApp \
  --settings KEY1=value1 KEY2=value2

# Enable logging
az webapp log config \
  --resource-group myResourceGroup \
  --name myWebApp \
  --web-server-logging filesystem

# Create deployment slot
az webapp deployment slot create \
  --resource-group myResourceGroup \
  --name myWebApp \
  --slot staging

# Swap slots
az webapp deployment slot swap \
  --resource-group myResourceGroup \
  --name myWebApp \
  --slot staging
```

## 🐳 Container Services

### Container Registry & ACI

Azure Container Registry:

```bash
# Create container registry
az acr create \
  --resource-group myResourceGroup \
  --name mycontainerregistry \
  --sku Basic

# Login to registry
az acr login --name mycontainerregistry

# Build image in ACR
az acr build \
  --registry mycontainerregistry \
  --image myapp:latest \
  .

# List images
az acr repository list \
  --name mycontainerregistry \
  --output table

# Push image to registry
docker tag myapp:latest mycontainerregistry.azurecr.io/myapp:latest
docker push mycontainerregistry.azurecr.io/myapp:latest
```

Container Instances:

```bash
# Create container instance
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mycontainerregistry.azurecr.io/myapp:latest \
  --cpu 1 \
  --memory 1 \
  --registry-login-server mycontainerregistry.azurecr.io \
  --registry-username username \
  --registry-password password

# List containers
az container list --output table

# Get container details
az container show \
  --resource-group myResourceGroup \
  --name mycontainer

# View container logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# Delete container
az container delete \
  --resource-group myResourceGroup \
  --name mycontainer --yes
```

## ⚙️ Kubernetes Service (AKS)

### Kubernetes Cluster Management

Create AKS cluster:

```bash
# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-managed-identity \
  --network-plugin azure \
  --network-policy azure

# List AKS clusters
az aks list --output table

# Get cluster credentials
az aks get-credentials \
  --resource-group myResourceGroup \
  --name myAKSCluster

# Scale cluster
az aks scale \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 5

# Upgrade cluster
az aks upgrade \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --kubernetes-version 1.24.0

# Delete cluster
az aks delete \
  --resource-group myResourceGroup \
  --name myAKSCluster --yes
```

AKS node management:

```bash
# Add node pool
az aks nodepool add \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name gpu-pool \
  --node-count 2 \
  --vm-set-type VirtualMachineScaleSets

# List node pools
az aks nodepool list \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --output table

# Scale node pool
az aks nodepool scale \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name gpu-pool \
  --node-count 3

# Delete node pool
az aks nodepool delete \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name gpu-pool
```

## 📊 Monitoring & Diagnostics

### Azure Monitor

Azure Monitor setup:

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group myResourceGroup \
  --workspace-name myWorkspace

# Create alert rule
az monitor metrics alert create \
  --resource-group myResourceGroup \
  --name myAlert \
  --description "CPU alert" \
  --scopes /subscriptions/{sub-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action add /subscriptions/{sub-id}/resourceGroups/myResourceGroup/providers/Microsoft.Insights/actionGroups/myActionGroup

# List alerts
az monitor metrics alert list --output table

# Create action group
az monitor action-group create \
  --resource-group myResourceGroup \
  --name myActionGroup

# Add email receiver
az monitor action-group update \
  --resource-group myResourceGroup \
  --name myActionGroup \
  --add-action email admin-email@example.com
```

Diagnostics & logging:

```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
  --name myDiagnostics \
  --resource /subscriptions/{sub-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --workspace /subscriptions/{sub-id}/resourcegroups/myResourceGroup/providers/microsoft.operationalinsights/workspaces/myWorkspace

# Query metrics
az monitor metrics list \
  --resource /subscriptions/{sub-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM

# Get activity log
az monitor activity-log list --resource-group myResourceGroup --output table
```

## 🔐 Identity & Access Management

### Azure AD & RBAC

User management:

```bash
# Create user
az ad user create \
  --display-name "John Doe" \
  --user-principal-name john@contoso.com \
  --password P@ssw0rd1234!

# List users
az ad user list --output table

# Get user details
az ad user show --id john@contoso.com

# Delete user
az ad user delete --id john@contoso.com
```

RBAC role assignment:

```bash
# List available roles
az role definition list --output table

# Assign role
az role assignment create \
  --assignee john@contoso.com \
  --role "Contributor" \
  --scope /subscriptions/{subscription-id}/resourceGroups/myResourceGroup

# List role assignments
az role assignment list \
  --resource-group myResourceGroup \
  --output table

# Remove role assignment
az role assignment delete \
  --assignee john@contoso.com \
  --role "Contributor" \
  --scope /subscriptions/{subscription-id}/resourceGroups/myResourceGroup
```

## 🔄 Azure DevOps & CI/CD

### Deployment & Automation

Azure Resource Manager (ARM):

```bash
# Deploy from template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file template.json \
  --parameters parameters.json

# Validate template
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file template.json

# Export resource group
az group export \
  --name myResourceGroup \
  > template.json

# Export single resource
az resource export \
  --resource-group myResourceGroup \
  --name myVM \
  --resource-type "Microsoft.Compute/virtualMachines"
```

Key Vault & secrets:

```bash
# Create key vault
az keyvault create \
  --resource-group myResourceGroup \
  --name myKeyVault

# Set secret
az keyvault secret set \
  --vault-name myKeyVault \
  --name mySecret \
  --value secretValue

# Get secret
az keyvault secret show \
  --vault-name myKeyVault \
  --name mySecret

# List secrets
az keyvault secret list \
  --vault-name myKeyVault \
  --output table

# Delete secret
az keyvault secret delete \
  --vault-name myKeyVault \
  --name mySecret
```

## 💰 Cost Management

### Billing & Optimization

Cost analysis:

```bash
# View current costs
az costmanagement query \
  --timeframe ActualLastMonth \
  --type "Usage" \
  --scope /subscriptions/{subscription-id}

# Create budget
az costmanagement budget create \
  --budget-name myBudget \
  --category Cost \
  --amount 1000 \
  --time-period start=2024-01-01,end=2024-12-31 \
  --scope /subscriptions/{subscription-id}

# List budgets
az costmanagement budget list --output table

# Set cost alert
az costmanagement alert create \
  --definition-id PreviewHighCost
```

## 📋 Azure Services Quick Reference

| Service | Purpose | CLI Command |
|---|---|---|
| Virtual Machines | IaaS compute | `az vm create` |
| App Service | PaaS web hosting | `az webapp create` |
| Azure Functions | Serverless | `az functionapp create` |
| Storage Account | Object storage | `az storage account create` |
| SQL Database | Relational DB | `az sql db create` |
| Cosmos DB | NoSQL database | `az cosmosdb create` |
| Virtual Network | Networking | `az network vnet create` |
| Load Balancer | Traffic distribution | `az network lb create` |
| AKS | Kubernetes | `az aks create` |
| Container Registry | Image registry | `az acr create` |
| Key Vault | Secrets management | `az keyvault create` |
| Azure Monitor | Monitoring | `az monitor create` |

## 🎓 Azure Common Patterns

### Web Application

- App Service
- SQL Database
- Storage Account
- App Insights
- CDN

### Microservices

- AKS
- API Management
- Service Bus
- Cosmos DB
- Azure Functions

### Data Analytics

- Data Lake
- Synapse
- Power BI
- Databricks
- HDInsight

### IoT & Edge

- IoT Hub
- IoT Central
- Azure Stack
- Event Grid
- Stream Analytics

### AI & ML

- Machine Learning
- Cognitive Services
- Bot Service
- Synapse ML
- Azure OpenAI

### Hybrid & Integration

- Azure Arc
- Azure Stack
- Integration Services
- Logic Apps
- Service Bus

## ✅ Azure Best Practices Checklist

### ✓ Subscription & Resource Management

- Organize with management groups
- Use resource groups logically
- Implement tagging strategy
- Enable cost management
- Set up budgets and alerts

### ✓ Security & Compliance

- Enable MFA for all users
- Use Key Vault for secrets
- Implement network security
- Enable Azure Security Center
- Regular security assessments

### ✓ Network & Infrastructure

- Use Virtual Networks properly
- Implement NSGs correctly
- Use private endpoints
- Enable DDoS protection
- Monitor network traffic

### ✓ Data & Storage

- Enable encryption at rest
- Use managed identities
- Regular backups
- Test disaster recovery
- Geo-redundancy where needed

### ✓ Application Design

- Use managed services
- Implement autoscaling
- Load balance traffic
- Health checks & monitoring
- Graceful degradation

## 💡 Cost Optimization

- Right-size resources regularly
- Use reserved instances
- Auto-shutdown unused resources
- Monitor spending daily
- Use Azure Hybrid Benefit

## ⚠️ Never

- Store credentials in code
- Skip backups/DR testing
- Leave default credentials
- Ignore security warnings
- Use root/admin for daily work

---

*Source: adapted from the Azure cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
