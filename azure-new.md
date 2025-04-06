## Azure PowerShell Cheatsheet

### Basic Commands
- **Authenticate an account**
```powershell
Connect-AzAccount
```
- **List authenticated accounts**
```powershell
Get-AzContext -ListAvailable
```
- **Set an account as default**
```powershell
Set-AzContext -SubscriptionId <SUBSCRIPTION_ID>
```
- **Remove a default account**
```powershell
Disconnect-AzAccount
```
- **Authenticate an Azure AD account**
```powershell
Connect-AzureAD
```
- **Get current Azure AD account**
```powershell
$res = Get-AzureADCurrentSessionInfo
$res
```
- **Authenticate via Microsoft Graph**
```powershell
Connect-MgGraph
```
- **Authenticate with service principal (App ID & Secret)**
```powershell
$SecurePassword = ConvertTo-SecureString -String "<AppSecret>" -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential("<AppId>", $SecurePassword)
Connect-AzAccount -ServicePrincipal -Tenant <TenantId> -Credential $Credential
```
- **Authenticate with service principal (App ID & Cert)**
```powershell
Connect-AzureAD -ApplicationId <AppId> -TenantId <TenantId> -CertificateThumbprint <Thumbprint>
```
- **Authenticate using access token**
```powershell
Connect-AzAccount -AccessToken <Token> -AccountId <AccountId>
```
- **Authenticate Azure AD using access token**
```powershell
Connect-AzureAD -AadAccessToken <Token> -TenantId <TenantId>
```

### General Enumeration
- **Enumerate tenant info**
```powershell
Invoke-WebRequest -Uri "https://login.windows.net/<domain>/.well-known/openid-configuration"
```
- **Check if Azure AD is used**
```powershell
Invoke-WebRequest -Uri "https://login.microsoftonline.com/getuserrealm.srf?login=<email>&xml=1"
```
- **Get Azure AD domains**
```powershell
Get-AzureADDomain
```
- **List Azure AD directory roles**
```powershell
Get-AzureADDirectoryRole
```
- **List users with a directory role**
```powershell
Get-AzureADDirectoryRoleMember -ObjectId <RoleId>
```
- **List all users**
```powershell
Get-AzureADUser
```
- **List all groups**
```powershell
Get-AzureADGroup
```
- **Get group owner**
```powershell
Get-AzureADGroupOwner -ObjectId <GroupId>
```
- **List applications**
```powershell
Get-MgApplication
```
- **Get application owner**
```powershell
Get-MgApplicationOwner -ApplicationId <AppId>
```
- **Get application permissions**
```powershell
$app = Get-MgApplication -ApplicationId <AppId>
$app.RequiredResourceAccess
```
- **Check Graph API role assignments**
```powershell
$res = Get-MgServicePrincipal -Filter "DisplayName eq 'Microsoft Graph'"
$res.AppRoles | Where-Object {$_.Id -eq '<RoleId>'}
```
- **Get service principal role assignments**
```powershell
Get-AzRoleAssignment -ObjectId <ObjectId>
```
- **Get role definition**
```powershell
Get-AzRoleDefinition -Name <RoleName>
```
- **List resources**
```powershell
Get-AzResource
```
- **List automation accounts**
```powershell
Get-AzAutomationAccount
```

### MicroBurst Enumeration
```powershell
Import-Module ./MicroBurst.psm1
Invoke-EnumerateAzureBlobs -Base "KEYWORD" -Verbose
Invoke-EnumerateAzureSubDomains -Base "KEYWORD" -Verbose
```

### VM Enumeration
- **List VMs**
```powershell
Get-AzVM | ConvertTo-Json
```
- **Run command on VM**
```powershell
Invoke-AzVMRunCommand -VMName <VMName> -ResourceGroupName <RGName> -CommandId RunShellScript -ScriptString "whoami"
```
- **Generate SSH key and add to VM**
```powershell
ssh-keygen
Invoke-AzVMRunCommand -VMName <VMName> -ResourceGroupName <RGName> -CommandId RunShellScript -ScriptString "echo 'ssh-rsa <Key>' >> /home/<User>/.ssh/authorized_keys"
```
- **Get VM network config**
```powershell
$vm = Get-AzVM -Name <VMName> -ResourceGroupName <RGName>
$vm.NetworkProfile
Get-AzPublicIpAddress -Name <PublicIPName>
Get-AzNetworkSecurityGroup -Name <NSGName>
```

### Privilege Escalation
- **Escalation via App Ownership**
```powershell
Add-MgApplicationPassword -ApplicationId <AppId>
```
- **Escalation via Automation Runbook**
```powershell
Import-AzAutomationRunbook -AutomationAccountName <Account> -ResourceGroupName <RGName> -Path ./Path/To/Script.ps1 -Name <RunbookName> -Published -Type PowerShell
Start-AzAutomationRunbook -AutomationAccountName <Account> -Name <RunbookName> -ResourceGroupName <RGName>
```

### Useful Tools
- MicroBurst: https://github.com/NetSPI/MicroBurst
- AADInternals: https://aadinternals.com/osint/
- Azure Storage Explorer