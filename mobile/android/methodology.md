---
title: Methodology
badge: Android
position: 1
---

## Getting Started

First of all verify that you have your Android phone plugged into the computer with debugging mode enabled. You can do so with the following command, checking if your device appears on the list:

```bash
adb devices
```

### Installing from application store

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

### Installing from APK file

When performing an assessment is possible that you were given the APK file from an alternative channel. You can install it on your phone with `adb install`:

```bash
adb install "{{ app-package com.luastan.app }}.apk
```

## Static analysis

Follow the tricks on the [Static Analysis](static) section. You can use the following checklist to verify you don't miss out on something:

### Static Checklist

- [ ] Run MobSF for quick wins
- [ ] Check if the code is obfuscated
- [ ] Check the manifest for missconfigurations
- [ ] Try to find sensitive information (unprotected api keys, hardcoded credentials, hidden URLs...) on the application sources

> TODO: Keep adding stuff

## Dynamic analysis

Follow the tricks on the [Dynamic Analysis](dynamic) section. You can use the following checklist to verify you don't miss out on something:

### Dynamic Checklist

- [ ] Check if there is Certificate pinning
- [ ] Check if dynamic instrumentation is possible and try to bypass it
- [ ] Try to bypass biometric authentication

> TODO: Keep adding stuff

