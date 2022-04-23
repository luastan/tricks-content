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

Get all users in the AAD

```powershell
Get-AzureADUser -All $true
```

Get all groups in the AD

```powershell
Get-AzureADGroup -All $true
```

Get all AD joined devices

```powershell
Get-AzureADDevice
```

Get Global administrators

```powershell
Get-AzureADDirectoryRole -Filter "DisplayName eq 'Global Administrator'" | Get-AzureADDirectoryRoleMember
```

Get Application Administrators

```powershell
Get-AzureADDirectoryRole -Filter "DisplayName eq 'Application Administrator'" | Get-AzureADDirectoryRoleMember
```

Get custom roles (This requires AzureADPreview Module)

```powershell
Get-AzureADMSRoleDefinition | ?{$_.IsBuiltin -eq $False} | select DisplayName
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