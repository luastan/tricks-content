---
title: Enumeration
description: Found or got access to any Azure account? Let's see what's next.
badge: Azure
---

Here are some of the ways you can enumerate Azure cloud environments once you get access to a valid account.
The following commands mostly use [az-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli), basic scripting commands for bash/powershell.
For bash scripts, the [jq](https://stedolan.github.io/jq/) is used a lot; install it from your distro packages:

<smart-tabs variable="distro" :tabs="{'arch': 'Arch linux', 'deb': 'Debian / Ubuntu / Kali'}">
<template v-slot:arch>

```bash
sudo pacman -Syu jq
```

</template>
<template v-slot:deb>

```bash
sudo apt update
sudo apt install jq
```

</template>
</smart-tabs>

There are a lot of cases where you want to add lines to a file if the line is not already in the file.
You can get fancy with bash/powershell but just using [anew](https://github.com/tomnomnom/anew) is much simpler.
Install it with `go install`:

```bash
go install -v github.com/tomnomnom/anew@latest
```

## Log-in

The first step is to log-in to azure. You can do this with azure's CLI:

```bash
az login -u "{{ username username }}" -p "{{ password PASS_OR_SECRET }}" --tenant "{{ tenant 12345-... }}" {{ exra-az-login-args --sp }}
```

## Subscriptions

An account (anything that can log-in to Azure: either a Service Principal or a user account) can have access from 0 to many Subscriptions. Those Subscriptions represent a collection of resource groups that at the same time are a subset of resources on the Azure tenant. Enumerating subscriptions is important to then be able to retrieve resources from each one.
A list of subscriptions for the accounts currently logged-in can be retrieved with the `az account list` command.

You will likelly want just to list the subscription ids.
You can do so with the following command:

<smart-tabs variable="bash-vs-powershell" :tabs="{'bash': 'Bash', 'powershell': 'Powershell'}">
<template v-slot:bash>

```bash
az account list | jq -r '.[].id' | sort | uniq
```

</template>
<template v-slot:powershell>

```powershell
# TODO: Implement the powershell command
```

</template>
</smart-tabs>

## KeyVault

Here is where the things start getting pretty wacky.
The following command wil list keyvaults for a subscription:

```bash
az keyvault list --subscription "{{ subscription 12345-... }}"
```

The idea here is to list and possibly dump every secret or key that you have access to on every KeyVault on every Subscription.

<smart-tabs variable="bash-vs-powershell" :tabs="{'bash': 'Bash', 'powershell': 'Powershell'}">
<template v-slot:bash>

```bash
for SUBSCRIPTION in $(az account list | jq -r '.[].id' | sort | uniq); do
    az keyvault list --subscription "$SUBSCRIPTION" | jq -r '.[].name | sort | uniq
done
```

</template>
<template v-slot:powershell>

```powershell
# TODO: Implement the powershell command
```

</template>
</smart-tabs>

From there you can obtain info for a specific keyvault with `az keyvault show`:

```bash
az keyvault show --name "{{ keyvault-name redacted_prod_kv_01 }}" --subscription "{{ subscription 12345-... }}"
```

### Enumerating IP restrictions

As KeyVaults hold pretty sensitive data, access to them should be restricted by IP.
You can list every KeyVault with no IP restriction with the following little script:

<smart-tabs variable="bash-vs-powershell" :tabs="{'bash': 'Bash', 'powershell': 'Powershell'}">
<template v-slot:bash>

```bash
for SUBSCRIPTION in $(az account list | jq -r '.[].id' | sort | uniq); do
    for KV in $(az keyvault list --subscription "$SUBSCRIPTION" | jq -r '.[].name | sort | uniq): do
        az keyvault show --name "$KV" --subscription "$SUBSCRITPION" | jq -r 'select(.properties.networkAcls == null or (.properties.networkAcls.defaultAction == "Allow")) | .id'
    done
done
```

</template>
<template v-slot:powershell>

```powershell
# TODO: Implement the powershell command
```

</template>
</smart-tabs>

### Secrets

KeyVaults have different kind of _"stores"_ or types of values that can store.

> TODO

## ACR

> TODO

## Graph Permissions

> TODO
