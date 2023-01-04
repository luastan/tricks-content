---
title: Checklist
description: What to do in case you have to hack an Azure Infrastructure
position: 1
---

# Checklist 

## 1 - Finding Creds

* [ ] Tokens from managed identity (all CKs)
* [ ] Powershell history (CK1)
* [ ] Secrets in vault (CK2)
* [ ] Password in userdata (CK2)
* [ ] Pass the PRT (CK2)
* [ ] Powershell transcripts (CK2)
* [ ] Add own pass to enterprise app (CK3)
* [ ] Vault pass (CK3)
* [ ] In public blob (CK4)
* [ ] Deployment template (CK4)
* [ ] Mimikatz (CK4)
* [ ] Create new user (CK1)

## 2 - Enumerate & Pivot

* [ ] Enumerate all resources
* [ ] Enumerate all users
* [ ] Check objects owned
* [ ] Check roles on automation account
* [ ] Check if webapp has managed identity
* [ ] Check permissions of managed identity
* [ ] Check which users/group has access to resource
* [ ] Check OS/IP of vm
* [ ] Check permission on specific resource
* [ ] Check if custom script extensions are installed
* [ ] List enterprise apps
* [ ] Check deployment template
* [ ] Check dynamic groups
* [ ] Check for application proxy
* [ ] Check users that can login to app proxy

## 3 - Bypasses

* [ ] Login with mobile user agent
* [ ] Change secondary email of guest to get in dynamic group
* [ ] Login to web portal
* [ ] Login to CLI

## 4 - Creds/Token usage

* [ ] Az Powershell
* [ ] AzureAD/ AzureAD Preview
* [ ] Az CLI
* [ ] Azure API
* [ ] AzPowershell