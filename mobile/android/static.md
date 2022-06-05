---
title: Static Analysis
badge: Android
description: Basic static analysis for android applications
position: 2
---

Let's say you want to analyse the <smart-variable variable="app-package">com.luastan.app</smart-variable> application. The following is a guide in order to provide some guildeness onto how to get started with Android applications static analysis. The required environment is described at the [Android page](/mobile/android#required-tools).


## Getting the APK

The first step is to locate the application file. We find it executing a shell command on the phone (If you don't know exactly the package name try the command with a string the app might have):

```bash
adb shell "pm list packages -f | grep {{ app-package com.luastan.app }} | sed 's/.*:\(.*apk\)=.*/\1/'"
```

Once you located the apk you can pull it (transfer it from your phone to your computer) with `adb pull`. You can use the _no-need-to-paste_ one-liner or specify the location:

<smart-tabs variable="adb-pull-fancy" :tabs="{'fancy': 'Fancy one liner', 'not-fancy': 'Paste the location'}">
<template v-slot:fancy>

```bash
adb pull $(adb shell "pm list packages -f | grep {{ app-package com.luastan.app }} | sed 's/.*:\(.*apk\)=.*/\1/'" | tr -d '\r\n') "{{ app-package com.luastan.app }}.apk"
```

</template>
<template v-slot:not-fancy>

```bash
adb pull "{{ app-location /data/app/.../base.apk }}" "{{ app-package com.luastan.app }}.apk"
```

</template>
</smart-tabs>

Both commands will pull the apk into <code><smart-variable variable="app-package">com.luastan.app</smart-variable>.apk</code> on your local machine.

## Unpackaging the APK

APK files are pretty much like zip archives full of compiled sources. You could just unzip an read the files, but is a better use of your time to unpack and decode the APK with `apktool`. You can extract/unpack/decode the <code><smart-variable variable="app-package">com.luastan.app</smart-variable>.apk</code> file you just pulled into a <code><smart-variable variable="app-package">com.luastan.app</smart-variable></code> directory with the following command:

```bash
apktool d "{{ app-package com.luastan.app }}.apk" -o "{{ app-package com.luastan.app }}"
```

Now you can explore the files at <code><smart-variable variable="app-package">com.luastan.app</smart-variable></code>. Start by taking a look at the `AndroidManifest.xml` file where the application is defined at <code><smart-variable variable="app-package">com.luastan.app</smart-variable>/AndroidManifest.xml</code>

## Nuclei

Nuclei can parse application files to find secrets and even misconfigurations. The default templates for this task belong to the `file` templates:

```bash
nuclei -t file -u "{{ app-package com.luastan.app }}" -me "{{ app-package nuclei_results_com.luastan.app }}"
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
