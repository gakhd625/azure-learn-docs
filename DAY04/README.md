# Azure Storage — Blob, File, SAS Tokens Day 4

## 1) Create a Storage Account (LRS)

### In Azure Portal:
- Go to **Storage accounts → Create**
- Fill:
  - Subscription: your lab subscription
  - Resource group: use the provided one (do NOT create new)
  - Storage account name: globally unique (e.g., storage1234501)
  - Region: same as lab
- Under **Redundancy**
  - Select **Locally-redundant storage (LRS)**
- Click **Review + Create → Create**

---

## 2) Upload Files (Portal + Azure CLI)

### A. Upload via Portal
- Open Storage Account
- Go to **Containers → + Container**
  - Name: `data`
  - Public access: Private
- Open container → Upload → select file

---

### B. Upload via PowerShell (Azure CLI)

#### Set variables
```powershell
$ACCOUNT_NAME="storage1234501"
$CONTAINER="data"
```

Create container (if needed)

```powershell
az storage container create `
  --name $CONTAINER `
  --account-name $ACCOUNT_NAME `
  --auth-mode login
```
Create test file

```powershell
"Hello Azure" | Out-File testfile.txt
```

Upload file

```powershell
az storage blob upload `
  --account-name $ACCOUNT_NAME `
  --container-name $CONTAINER `
  --name testfile.txt `
  --file .\testfile.txt `
  --auth-mode login
```

## 3) Generate SAS Token + Test Expiry
In Azure Portal:
- Go to Storage Account → Shared access signature
- Set:
    - Services: Blob
    - Resource types: Container + Object
    - Permissions: Read, Write, List
    - Expiry: 10–15 minutes
    - Click Generate SAS

## Test SAS Important
Use container or file URL
```powershell
https://storage1234501.blob.core.windows.net/data?SAS_TOKEN
```
OR
```powershell
https://storage1234501.blob.core.windows.net/data/testfile.txt?SAS_TOKEN
```
Expected:
- Before expiry → works
- After expiry → AuthenticationFailed

## 4) Create Azure File Share + Mount
### A. Create File Share
- Go to Storage Account → File shares → + File share
- Name: fileshare1
- Quota: default
### B. Mount File Share (PowerShell)
- Get storage key
```powershell
$KEY=$(az storage account keys list `
  --account-name $ACCOUNT_NAME `
  --query "[0].value" -o tsv)
```
Map drive
```powershell
net use Z: \\storage1234501.file.core.windows.net\fileshare1 /u:storage1234501 
```
TEST
```powershell
"Test file" | Out-File Z:\test.txt 
```
## 5) Lifecycle Policy (Move to Cool after 30 days)
### In Azure Portal:
- Go to Storage Account → Lifecycle management → Add rule
### Configure:
- Name: move-to-cool
- Scope: Blob
- Filter: default (all blobs)
### Action:
- Base blobs:
    - Move to Cool storage
    - After 30 days since modification
### Final Checklist
- Storage account (LRS) created
- Blob container created
- File uploaded via portal and CLI
- SAS tested (works + expires)
- File share created
- Z: drive mounted
- File visible in Azure
- Lifecycle policy applied
### Notes
- Use provided resource group (sandbox restriction)
- "created": false = container already exists (OK)
- SAS errors:
    - AuthenticationFailed = expired (expected)
    - InvalidQueryParameterValue = incorrect UR