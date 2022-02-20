---
title: Azure
badge: Cloud Pentesting
description: Microsoft Azure, often referred to as Azure, is a cloud computing service operated by Microsoft for application management via Microsoft-managed data centers.
---

## Starting point !

The following steps should guide you through Azure:

1. Understanding what Azure and Azure AD are, how Microsoft envisions it and the basic concepts around them: [Azure and Azure AD](/cloud/azure#azure-and-azure-ad)
2. Learning the different official ways of [interacting with Azure](/cloud/azure#interacting-with-azure):
   - [Azure portal](/cloud/azure#portal): The basic one through a gragical interface on Microsoft's site.
   - [Microsoft Graph](/cloud/azure#microsoft-graph): An API to interact with Azure AD specifically.
   - [Azure-cli](/cloud/azure#azure-cli): Command line tool to interact with Azure and Azure AD
   - [Powershell Modules](/cloud/azure#powershell-modules): A subset of modules for powershell for accessing Azure / Azure AD
   - [Exchange API](/cloud/azure#exchange): An API to interact with Azure AD. Deprecated in favour of Microsoft Graph.
   - [Office 365 APIs](/cloud/azure#office-365-apis): A subset of APIs to interact with some Office 365. Deprecated in favour of Microsoft Graph




## Azure Active Directory (Azure AD or AAD)


## Interacting with Azure


### Portal

Access the portal here: [http://portal.azure.com/](http://portal.azure.com)\
To start the tests you should have access with a user with **Reader permissions over the subscription** and **Global Reader role in AzureAD**. If even in that case you are **not able to access the content of the Storage accounts** you can fix it with the **role Storage Account Contributor**.

Remember that if the **Security Centre Standard Pricing Tier** is being used and **not** the **free** tier, you can **generate** a **CIS compliance scan report** from the azure portal. Go to _Policy & Compliance -> Regulatory Compliance_ (or try to access [https://portal.azure.com/#blade/Microsoft\_Azure\_Security/SecurityMenuBlade/22](https://portal.azure.com/#blade/Microsoft\_Azure\_Security/SecurityMenuBlade/22)).

> If the company is not paying for a Standard account you may need to review the **CIS Microsoft Azure Foundations Benchmark** by "hand" (you can get some help using the following tools). Download it from [**here**](https://www.newnettechnologies.com/cis-benchmark.html?keyword=\&gclid=Cj0KCQjwyPbzBRDsARIsAFh15JYSireQtX57C6XF8cfZU3JVjswtaLFJndC3Hv45YraKpLVDgLqEY6IaAhsZEALw\_wcB#microsoft-azure).

### Azure-cli

It is recommended to **install azure-cli** in a **linux** and **windows** virtual machines (to be able to run powershell and python scripts): [https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)\
Then, run `az login` to login. Note the **account information** and **token** will be **saved** inside _\<HOME>/.azure_ (in both Windows and Linux).


### Powershell modules

There is a variety of Powershell modules to interact with Azure 


#### MSOnline

```powershell
Install-Module MSOnline
```


### Microsoft Graph



### Exchange

> It is deprecated but in the meantime could be useful


### Office 365 APIs

Subset of APIS