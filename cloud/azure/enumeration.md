---
title: Enumeration
description: Found or got access to any Azure account? Let's see what's next.
badge: Azure
---


## Log-in
The first step is to log-in to azure. You can do this with azure's CLI:

```bash
az login -u "{{ username username }}" -p "{{ password PASS_OR_SECRET }}" --tenant "{{ tenant 12345-... }}" {{ exra-az-login-args --sp }}
```


## Subscriptions
An account (anything that can log-in to Azure: either a Service Principal or a user account) can have access from 0 to many Subscriptions. Those Subscriptions represent a collection of resource groups that at the same time are a subset of resources on the Azure tenant. Enumerating subscriptions is important to then be able to retrieve resources from each one.


## KeyVault


## ACR


## Graph Permissions

