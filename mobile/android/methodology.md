---
title: Methodology
badge: Android
position: 2
---

## Getting Started

First of all verify that you have your Android phone plugged into the computer with debugging mode enabled. You can do so with the following command, checking if your device appears on the list:

```bash
adb devices
```

### Installing the app

Install it from your desired source. It is possible that you were given the APK file from an alternative channel. You can install it from your PC with `adb install`:

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

