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

List KeyVaults

```powershell
Get-AzKeyVault
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

List AAD Apps

```powershell
az ad app list
# Show information about the application
az ad app show --id {{ ApplicationId appid }}
# List owners of application
az ad app owner list --id {{ ApplicationId appid }} --query "[].[displayName]"
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

## Using APIs

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