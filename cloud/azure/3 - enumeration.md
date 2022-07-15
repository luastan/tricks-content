---
title: Enumeration
description: Finally we have a user, we have one foot inside, what can we do?
position: 3
---

# Enumeration

## Azure AD Powershell module

Azure AD is one of the weirdest Powershell modules that exists. It has native dlls that makes it platform dependant to windows. Moreover it requires the dotnet version to be amd64 so you have to launch a new PowerShell x64 window.

Login: 

```powershell
$passwd = ConvertTo-SecureString '{{ Password password }}' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ('{{ Username user@domain.com }}', $passwd)
Connect-AzureAD -Credential $creds
```

Current session info:

```powershell
Get-AzureADCurrentSessionInfo
```

Get all users in the AAD

```powershell
Get-AzureADUser -All $true
# Search string in DisplayName and userPrincipalName, no wildcards allowed
Get-AzureADUser -SearchString "{{ Filter string }}"
# Search using powershell
Get-AzureADUser -All $true | ?{$_.Displayname -match "{{ Filter string }}"}
# List attributes of user
Get-AzureADUser -ObjectId '{{ Username user@domain.com }}' | fl *

# Search all attributes that match password
Get-AzureADUser -All $true | %{$Properties = $_; $Properties.PSObject.Properties.Name | %{if ($Properties.$_ -match 'password') {"$($Properties.UserPrincipalName) - $_ - $($Properties.$_)"}}}

# Users synced from on-prem
Get-AzureADUser -All $true | ?{$_.OnPremisesSecurityIdentifier -ne $null}
# Users synced from Azure
Get-AzureADUser -All $true | ?{$_.OnPremisesSecurityIdentifier -eq $null}
```

Objects created from a specific user

```powershell
Get-AzureADUserOwnedObject -ObjectId '{{ Username user@domain.com }}'
```

Get all groups in the AD

```powershell
Get-AzureADGroup -All $true
# Filter DisplayName, no wildcards allowed
Get-AzureADGroup -SearchString '{{ Filter string }}' | fl *

# Groups that allow dynamic membership
Get-AzureADMSGroup | ?{$_.GroupTypes -eq 'DynamicMembership'}

# Groups synced from onprem
Get-AzureADGroup -All $true | ?{$_.OnPremisesSecurityIdentifier -ne $null}
```

Get members of a group

```powershell
Get-AzureADGroupMember -ObjectId {{ Object object}}
```

Get all AD joined devices

```powershell
Get-AzureADDevice -All $true | fl *
Get-AzureADDeviceConfiguration | fl *
# Devices owned by user
Get-AzureADUserOwnedDevice -ObjectId {{ Username user@domain.com }}
```

Get users with role:

```powershell
Get-AzureADDirectoryRole -Filter "DisplayName eq '{{ Role role }}'" | Get-AzureADDirectoryRoleMember
```

Interesting roles:
* Global Administrator
* Application Administrator

Get custom roles (This requires AzureADPreview Module)

```powershell
Get-AzureADMSRoleDefinition | ?{$_.IsBuiltin -eq $False} | select DisplayName
```

Get application objects

```powershell
Get-AzureADApplication -All $true
Get-AzureADApplication -ObjectId {{ ApplicationId appid }}
Get-AzureADApplication -All $true | ?{$_.DisplayName -match "{{ Displayname displayname }}"}

# Applications with an application password but password value not shown
Get-AzureADApplicationPasswordCredential

# Get owner of application
Get-AzureADApplication -ObjectId {{ ApplicationId appid}} | Get-AzureADApplicationOwner |fl *
```

Get service principals (Enterprise applications)

```powershell
Get-AzureADServicePrincipal -All $true
Get-AzureADServicePrincipal -ObjectId {{ ApplicationId appid }}
Get-AzureADServicePrincipal -All $true | ?{$_.DisplayName -match "{{ Displayname displayname }}"}

# Get owner of application
Get-AzureADServicePrincipal -ObjectId {{ ApplicationId appid }} | Get-AzureADApplicationOwner |fl *
# Get objects owned by a service principal
Get-AzureADServicePrincipal -ObjectId {{ ApplicationId appid }}  | Get-AzureADServicePrincipalOwnedObject
# Get group and role membership of a service principal
Get-AzureADServicePrincipal -ObjectId {{ ApplicationId appid }} | Get- AzureADServicePrincipalMembership |fl *
```

Check if a compromised host has an ADSyncConnector:

```powershell
Get-ADSyncConnector
```

## Az Powershell module

The Az PowerShell module is a set of cmdlets for managing Azure resources directly from PowerShell. PowerShell provides powerful features for automation that can be leveraged for managing your Azure resources, for example in the context of a CI/CD pipeline.

The Az PowerShell module is the replacement of AzureRM and is the recommended version to use for interacting with Azure.

> Note: To read this the user must have at least one subscription

Login: 

```powershell
$passwd = ConvertTo-SecureString '{{ Password password }}' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ('{{ Username user@domain.com }}', $passwd)
Connect-AzAccount -Credential $creds
```

Get data of the context

```powershell
Get-AzContext
Get-AzContext -ListAvailable
Get-AzSubscription
Get-AzRoleAssignment
```

Get accesible resources

```powershell
Get-AzResource
```

Get all the role assignments for an user

```powershell
Get-AzRoleAssignment -SignInName {{ Username user@domain.com }}

# Get role assignment for the current user to a specific resource
Get-AzRoleAssignment -Scope {{ ResourceID /subscriptions/e149bb0a-224a-4802-8ef3-7477110706ad/resourceGroups/cloud-shell-storage-centralindia/providers/Microsoft.Automation/automationAccounts/SecureAutomation/runbooks/CreateResourceGroup }}

# Do it for all resources
Get-AzResource | %{try{Get-AzRoleAssignment -Scope $_.ResourceId}catch{}}

```

Get Azure VMs

```powershell
Get-AzVM | fl
```

List Web App Services

```powershell
Get-AzWebApp | ?{$_.Kind -notmatch "functionapp"}
```

List FunctionApps

```powershell
Get-AzFunctionApp
```

List storage accounts

```powershell
Get-AzStorageAccount | fl
```

List automation accounts

```powershell
Get-AzAutomationAccount

# List runbooks
Get-AzAutomationRunbook -AutomationAccountName "{{ AutomationAccountName SecureAutomation }}" -ResourceGroupName "{{ ResourceGroupName cloud-shell-storage-centralindia }}"
Get-AzAutomationRunbook -AutomationAccountName "{{ AutomationAccountName SecureAutomation }}" -ResourceGroupName "{{ ResourceGroupName cloud-shell-storage-centralindia }}" -Name {{ FunctionName CreateFunctionApp }}

# Export runbook
Export-AzAutomationRunbook -AutomationAccountName "{{ AutomationAccountName SecureAutomation }}" -ResourceGroupName "{{ ResourceGroupName cloud-shell-storage-centralindia }}" -Name {{ FunctionName CreateFunctionApp }} -Slot "Published" -OutputFolder ".\runbooks"

# Dump all runbooks
Get-AzAutomationRunbook -AutomationAccountName "{{ AutomationAccountName SecureAutomation }}" -ResourceGroupName "{{ ResourceGroupName cloud-shell-storage-centralindia }}" | %{Export-AzAutomationRunbook -AutomationAccountName $_.AutomationAccountName -ResourceGroupName $_.ResourceGroupName -Name $_.Name -Slot "Published" -OutputFolder ".\runbooks"}
```

List KeyVaults

```powershell
Get-AzKeyVault

# List secrets in keyvault
Get-AzKeyVaultSecret -VaultName {{ VaultName vaultname }}

# Access keyvault
Get-AzKeyVaultSecret -VaultName {{ VaultName vaultname }} -Name {{ SecretName secretname }}-AsPlainText
```

## AZ Cli

The Azure Command-Line Interface (CLI) is a cross-platform command-line tool to connect to Azure and execute administrative commands on Azure resources. It allows the execution of commands through a terminal using interactive command-line prompts or a script.

> Note: To read this the user must have at least one subscription

Login: 

```powershell
az login -u '{{ Username user@domain.com }}' -p '{{ Password password }}'
az login -u '{{ Username user@domain.com }}' -p '{{ Password password }}' --allow-no-subscriptions
```

Find information about the current user

```powershell
az account tenant list
az account subscription list
az ad signed-in-user show
```

List AAD users

```powershell
az ad user list
```

List AAD Groups

```powershell
az ad group list
az ad group show -g "{{ Group VM Admins or Id }}"
# Groups that contain admin
az ad group list --query "[?contains(displayName,'admin')].displayName"
# Groups synced from on-prem
az ad group list --query "[?onPremisesSecurityIdentifier!=null].displayName"
# Groups created in azure
az ad group list --query "[?onPremisesSecurityIdentifier==null].displayName"

# Members of a group
az ad group member list -g "{{ Group VM Admins or Id }}" --query "[].[displayName]"
```

List AAD Apps

```powershell
az ad app list
# Show information about the application
az ad app show --id {{ ApplicationId appid }}
# List owners of application
az ad app owner list --id {{ ApplicationId appid }} --query "[].[displayName]"
```

List Service Principals

```powershell
az ad sp list

az ad sp list --all

# Show information about the application
az ad sp show --id {{ ApplicationId appid }}
# List owners of application
az ad sp owner list --id {{ ApplicationId appid }} --query "[].[displayName]"
# Service principals owned by current user
az ad sp list --show-mine

# Service principals that have password credentials
az ad sp list --all --query "[?passwordCredentials != null].displayName"
# Service principals that have key credentials
```

Export configuration for Service Principals

```powershell

# List keys
az appconfig kv list --connection-string "Endpoint={{ ConnectionString https://escrow3.azconfig.io;Id=q63q-lab-s0:IkiICUfj7aoiWdfW+fLf;Secret=j50xk3bTCW3uZ4R+jMAyu/feCaKRXrAucE5dJs5OHmY= }}"

# Export key-value
az appconfig kv export --connection-string "Endpoint={{ ConnectionString https://escrow3.azconfig.io;Id=q63q-lab-s0:IkiICUfj7aoiWdfW+fLf;Secret=j50xk3bTCW3uZ4R+jMAyu/feCaKRXrAucE5dJs5OHmY= }}" --key "{{ Key Escrow3App:Settings:Passwd }}" -d file --path appconfig.json --format json
```

List owned objects

```powershell
az ad signed-in-user list-owned-objects
```

List vms

```powershell
az vm list
```

List WebApps

```powershell
az webapp list
```

List FunctionApps

```powershell
az functionapp list 
```

List Storage Accounts

```powershell
az storage account list
```

List KeyVaults

```powershell
az keyvault list
```

List Resources

```powershell
# First list locations if you do not know where the resources are
# az account list-locations
az resource list --location westus
```

## Using APIs

### Getting a token

Most of the above modules (Az, Powershell and AzureAD) allow to create access tokens for authentication.

#### Azure Az Powershell module

Get token for resource manager (ARM)

```powershell
(Get-AzAccessToken).Token
```

Request a token for any other service principal: AAD Graph, AnalysisServices, Arm, Attestation, Batch, DataLake, KeyVault, OperationalInsights, ResourceManager, Synapse

```powershell
Get-AzAccessToken -ResourceTypeName {{ ResourceTypeName AadGraph }}
(Get-AzAccessToken -Resource "{{ Resource_URI https://graph.microsoft.com }}").Token
```

Using the token is also simple

```powershell
Connect-AzAccount -AccountId {{ Username user@domain.com }} -AccessToken {{ Access_Token eyJ0eXA... }}
Connect-AzAccount -AccountId {{ Username user@domain.com }} -AccessToken {{ Access_Token eyJ0eXA... }} -GraphAccessToken {{ Graph_Token eyJ0eXA... }}
Connect-AzureAD -AccountId
{{ Username user@domain.com }} -AadAccessToken {{ Access_Token eyJ0eXA... }}
```

#### AZ Cli

Request a token for ARM

```powershell
az account get-access-token
```

Request a token for aad-graph, arm, batch, data-lake, media, ms-graph, oss-rdbms

```powershell
az account get-access-token --resource-type {{ Resource-type ms-graph }}
az account get-access-token --resource {{ Resource_URI https://graph.microsoft.com }}
```

### ARM (Azure Resource Manager) (management.azure.com)

```powershell
# List subscriptions
$Token = '{{ Access_token eyJ0eXA...}}'
$URI = 'https://management.azure.com/subscriptions?api-version=2020-01-01'
$RequestParams = @{
    Method  = 'GET'
    Uri = $URI Headers = @{
    'Authorization' = "Bearer $Token"
    }
}
(Invoke-RestMethod @RequestParams).value

# List accessible resources
$Token = '{{ Access_token eyJ0eXA...}}'
$URI = 'https://management.azure.com/subscriptions/{{ Subscription subscription }}/resources?api-version=2020-10-01'
$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token"
    }
}
(Invoke-RestMethod @RequestParams).value

# List actions allowed for a resource
$Token = '{{ Access_token eyJ0eXA...}}'
$URI = 'https://management.azure.com/{{ ResourceId subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{name}/providers/Microsoft.Authorization/permissions }}?api-version=2015-07-01'
$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token"
    }
}
(Invoke-RestMethod @RequestParams).value
```

### MS-Graph (graph.microsoft.com)

```powershell
# List users
$Token = '{{ Access_token eyJ0eXA...}}'
$URI = 'https://graph.microsoft.com/v1.0/users'
$RequestParams = @{
    Method  = 'GET'
    Uri = $URI Headers = @{
    'Authorization' = "Bearer $Token"
    }
}
(Invoke-RestMethod @RequestParams).value
```

```powershell
# List enterprise applications
$Token = '{{ Access_token eyJ0eXA...}}'
$URI = 'https://graph.microsoft.com/v1.0/servicePrincipals'
$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $Token"
    }
}
(Invoke-RestMethod @RequestParams).value
```

```powershell
# List applications
$token = (Get-AzAccessToken -ResourceUrl 'https://graph.microsoft.com').Token
$URI = 'https://graph.microsoft.com/v1.0/applications'
$RequestParams = @{
    Method  = 'GET'
    Uri     = $URI
    Headers = @{
        'Authorization' = "Bearer $token"
    }
}
(Invoke-RestMethod @RequestParams).value
```

## Dump everything using [RoadRecon](https://github.com/dirkjanm/ROADtools)

ROADtools is a framework to interact with Azure AD. It currently consists of a library (roadlib) and the ROADrecon Azure AD exploration tool.

Gather:

```powershell
roadrecon auth -u '{{ Username user@domain.com }}' -p '{{ Password password }}'
roadrecon gather
```

Gui:

```powershell
roadrecon gui
```