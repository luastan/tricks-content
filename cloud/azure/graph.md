---
title: Microsoft Graph
description: 
position: 1
badge: Azure
---


## Microsoft Graph permissions

Understanding permissions is key to understand how to perform privilege escalation.

### Delegated and application permissions

Microsoft Graph has two types of permissions:

- **Delegated permissions** are used by apps that have a signed-in user present. For these apps, either the user or an administrator consents to the permissions that the app requests and the app can act as the signed-in user when making calls to Microsoft Graph. Some delegated permissions can be consented by non-administrative users, but some higher-privileged permissions require [administrator consent](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes#using-the-admin-consent-endpoint).

- **Application permissions** are used by apps that run without a signed-in user present. For example, apps that run as background services or daemons. Application permissions can only be [consented by an administrator](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-scopes#requesting-consent-for-an-entire-tenant).

*Effective permissions* are the permissions that your app has when making requests to Microsoft Graph. It's important to understand the difference between the delegated and application permissions that your app is granted, and its effective permissions when making calls to Microsoft Graph.

-   For delegated permissions, the effective permissions of your app are the intersection of the delegated permissions the app has been granted (via consent) and the privileges of the currently signed-in user. Your app can never have more privileges than the signed-in user. Within organizations, the privileges of the signed-in user are determined by policy or by membership in one or more administrator roles. For more information about administrator roles, see [Assigning administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-assign-admin-roles).

    For example, assume your app has been granted the *User.ReadWrite.All* delegated permission. This permission nominally grants your app permission to read and update the profile of every user in an organization. If the signed-in user is a global administrator, your app can update the profile of every user in the organization. However, if the signed-in user isn't in an administrator role, your app can update only the profile of the signed-in user. It won't update the profiles of other users in the organization because the signed-in user doesn't have those privileges.

-   For application permissions, the effective permissions of your app will be the full level of privileges implied by the permission. For example, an app that has the *User.ReadWrite.All* application permission can update the profile of every user in the organization.

![Microsoft Graph exposes delegated and application permissions but authorizes requests based on the app's effective permissions.](https://docs.microsoft.com/en-us/graph/images/auth-v2/permission-types.png)

> Note By default, apps that have been granted application permissions to the following data sets can access all the mailboxes in the organization:

-   [Calendars](https://docs.microsoft.com/en-us/graph/permissions-reference#calendars-permissions)
-   [Contacts](https://docs.microsoft.com/en-us/graph/permissions-reference#contacts-permissions)
-   [Mail](https://docs.microsoft.com/en-us/graph/permissions-reference#mail-permissions)
-   [Mailbox settings](https://docs.microsoft.com/en-us/graph/permissions-reference#mail-permissions)

> Administrators can configure [application access policy](https://docs.microsoft.com/en-us/graph/auth-limit-mailbox-access) to limit app access to *specific* mailboxes.

For a complete list of delegated and application permissions for Microsoft Graph, and which permissions require administrator consent, see the [Permissions reference](https://docs.microsoft.com/en-us/graph/permissions-reference).