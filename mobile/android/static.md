---
title: Static Analysis
badge: Android
description: Basic static analysis for android applications
position: 2
---

Let's say you want to analyse the <smart-variable variable="app-package">com.luastan.app</smart-variable> application. The following is a guide in order to provide some guildeness onto how to get started with Android applications static analysis. The required environment is described at the [Android page](/mobile/android#required-tools).

## Unpackaging the APK

APK files are pretty much like zip archives full of compiled sources. You could just unzip an read the files, but is a better use of your time to unpack and decode the APK with `apktool`. You can extract/unpack/decode the <code><smart-variable variable="app-package">com.luastan.app</smart-variable>.apk</code> file you just pulled into a <code><smart-variable variable="app-package">com.luastan.app</smart-variable></code> directory with the following command:

```bash
apktool d "{{ app-package com.luastan.app }}.apk" -o "{{ app-package com.luastan.app }}"
```

Now you can explore the files at <code><smart-variable variable="app-package">com.luastan.app</smart-variable></code>. Start by taking a look at the `AndroidManifest.xml` file where the application is defined at <code><smart-variable variable="app-package">com.luastan.app</smart-variable>/AndroidManifest.xml</code>

## Nuclei

Nuclei can parse application files to find secrets and even misconfigurations. The default templates for this task belong to the `file` templates:

```bash
nuclei -t file -u "{{ app-package com.luastan.app }}"
```
> To be continued ...

## Keytest
With [Keytest](https://github.com/luastan/keytest) you can find and check API keys that might be included with the APP sources:

You can find the API keys with:
```bash
keytest find "{{ app-package com.luastan.app }}"
```

Or even directly test the API keys to check if any is vulnerable to some kind of misconfiguration:

```bash
keytest check "{{ app-package com.luastan.app }}" -o "{{ app-package com.luastan.app }}.apikeys.md"
```

## Decompiling into Java source

## Mobile Security Framework (MobSF)
