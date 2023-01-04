---
title: Snooze's Environment
description: Things I use and how I use them
---

# Windows

## Scoop

Scoop allows to install apps in userland using terminal

```powershell
# Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex
```

Then using scoop we can find lots of utils

```powershell
scoop install git curl jq python go nodejs
```

# MacOS

## Run Powershell x64

```bash
arch -x86_64 pwsh
# Using this most of the azure commands do work
```

# Kali

# Cloud

# Hardware and ergonomics

# Powershell

Clear the history in powershell every once in a while

```powershell
Clear-History
```

List installed modules and uninstall

```powershell
Get-InstalledModule | grep AzureAD
```