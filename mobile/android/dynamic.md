---
title: Dynamic analysis
badge: Android
description: Some tricks for Android dynamic analysis
position: 3
---

## Certificate Pinning

Intercepting traffic from the application is one of the first things you want to do. Start by setting your burp as a proxy for the phone.

### Android 7+ issues

On Android devices starting from version 7, the OS forces the server certificate to be signed by a trusted CA.
This means you will need to add your own CA certificate onto Android's system certificate store.

### Bypass Certificate pinning

If you can intercept HTTPS traffic from the phone web browser but the app does not work and/or shows errors, the app most likely implements certificate pinning.
Fortunately there is a bunch of ways you can bypass this protection.

#### Using Objection

One of the most simple ways to bypass the protection is just by using the `android sslpinning disable` command with Objection:

```shell
objection -g "{{ app-package com.luastan.app }}"
```