---
title: Common Azure leaks
description: Due to the dynamic nature of cloud environments, secrets and other kinds of sensible information might get stored in different kinds of metadata.
badge: Azure
position: 11
---


## Secrets inside Azure AD User Attributes
On Azure AD environments is common to find credentials (and other hidden gems) in description or comment fields for users.

### Using MSOnline
First import the `MSOnline` module and connect to the service if you haven't already: 

```powershell
Import-Module MSOnline
Connect-MsolService
```

You can find those hidden gems with the following script:

```powershell[Get-SecretsInUserProps.ps1]
$users = Get-MsolUser
foreach($user in $users){
    $props = @()
    $user | Get-Member | foreach-object {
        $props += $_.Name
    }
    foreach($prop in $props){
        if($user.$prop -like "{{ secret *password* }}"){
            Write-Output ("[+] " + $user.UserPrincipalName + "[" + $prop + "]" + ": " + $user.$prop)
        }
    }
}
```

### Using Graph


> TODO: A curl script or a go thing to iterate over every user we can access to list every single property. It is probably a better idea to just make something to dump the whole thing into a json and grep it afterwards with gron or smthing

