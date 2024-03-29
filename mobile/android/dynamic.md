---
title: Dynamic analysis
badge: Android
description: Some tricks for Android dynamic analysis
position: 4
---


## Dynamic Instrumentation Basics

### Frida

<smart-tabs variable="frida" :tabs="{'no-script': 'Without script', 'with-script': 'With Script'}">
<template v-slot:no-script>

```shell
frida -U -f "{{ app-package com.luastan.app }}"
```

</template>
<template v-slot:with-script>

```shell
frida -U -f "{{ app-package com.luastan.app }}" -l "{{ frida-script anti-root.js }}"
```

</template>
</smart-tabs>

#### Locate a running app

Look for a running app. I make use of the following command more often than expected:

<smart-tabs variable="awk" :tabs="{'awk': 'awk', 'alternatives': 'Alternatives'}">
<template v-slot:awk>

```shell
frida-ps -U | grep -i "{{ app-name MyApp }}" | awk '{print $1}'
```

</template>
<template v-slot:alternatives>

If you don't have `awk` you can use one of the following `xargs`+`cut` combinations:

```shell
frida-ps -U | grep -i "{{ app-name MyApp }}" | xargs | cut -d ' ' -f1
frida-ps -U | grep -i "{{ app-name MyApp }}" | xargs echo | cut -d ' ' -f1
frida-ps -U | grep -i "{{ app-name MyApp }}" | {{ trim-whitespaces-command xargs echo }} | cut -d ' ' -f1
frida-ps -U | grep -i "{{ app-name MyApp }}" | cut -d ' ' -f{{ cut-field 1 }}
```

</template>
</smart-tabs>

### Objection

<smart-tabs variable="objection-mode" :tabs="{'launch': 'Launch app', 'attatch': 'Attatch to app'}">
<template v-slot:launch>

```shell
objection -g "{{ app-package com.luastan.app }}" explore
```

</template>
<template v-slot:attatch>

- Without knowing the PID:

```shell
objection -g $(frida-ps -U | grep -i "{{ app-name MyApp }}" | awk '{print $1}') explore
```

- Knowing the PID:

```shell
objection -g {{ PID 123 }} explore
```

</template>
</smart-tabs>

#### Hooking helpers

While JadX provides nice help to hook with Frida, you might find the following quite usefull to look for **loaded** classes.

- Search classess:

```shell
android hooking search classes {{ class-search com.root.detection }}
```

- List **loaded** classess:

```shell
android hooking list classes
```

- Search methods:

```shell
android hooking search methods {{ package-name com.luastan.app }} {{ classname RootDetection }}
```

- List methods with parameters:

```shell
android hooking list class_methods {{ class com.root.detection }}
```

## Certificate Pinning

Intercepting traffic from the application is one of the first things you want to do. Start by setting your burp as a proxy for the phone.

### Android 7+ issues

On Android devices starting from version 7, the OS forces the server certificate to be signed by a trusted CA.
This means you will need to add your own CA certificate onto Android's system certificate store.
You can copy it manually or automatically with a Magisk module as shown [here](/mobile/android/provisioning#system-ca).

### Bypass Certificate pinning

If you can intercept HTTPS traffic from the phone web browser but the app does not work and/or shows errors, the app most likely implements certificate pinning.
Fortunately there is a bunch of ways you can bypass this protection.

#### Using Objection

One of the most simple ways to bypass the protection is just by using the `android sslpinning disable` command with Objection:

```shell
objection -g "{{ app-package com.luastan.app }}" -s "android sslpinning disable"
```

## Keystore

### WithSecure Labs scripts

This is a very useful collection of frida scripts that helps a lot when auditing the keystore usage of an app.
You can find the scripts on [their Github repo](https://github.com/WithSecureLabs/android-keystore-audit/tree/master/frida-scripts), and a nice article on [their blog](https://labs.withsecure.com/publications/how-secure-is-your-android-keystore-authentication).

#### Tracer

The tracing script is called `tracer-keystore.js` and helps a lot when reverse engineering the keystore usage.
Load it with:

```shell
frida -U -f "{{ app-package com.luastan.app }}" --no-pause -l "{{ frida-scripts-location frida-scripts/ }}tracer-keystore.js"
```

If you [read the script](https://github.com/WithSecureLabs/android-keystore-audit/blob/master/frida-scripts/tracer-keystore.js#L7) you can find more info about the utilities that come with it:

- `KeystoreListAllAliases()`, `ListAliasesStatic()`:
  - List all aliases in keystores of known hardcoded types (in keystoreTypes)
- `KeystoreListAllAliasesOnAllInstances()`, `ListAliasesRuntime()`
  - List all aliases in keystores of all instances obtained during app runtime.
  - Instances that will be dumped are collected via hijacking
  - `Keystore.getInstance()` -> `hookKeystoreGetInstance()`
- `ListAliasesAndroid()`
  - List all aliases in AndroidKeyStore.
- `ListAliasesType(String type)`
  - List all aliases in a keystore of given 'type'.
  - Example: `ListAliasesType('AndroidKeyStore')`;
- `ListAliasesObj(Object obj)`
  - List all aliases for a given keystore object. 
  - Example: `ListAliasesObj(keystoreObj)`;
- `GetKeyStore(String name)`
  - Retrieve keystore instance from keystoreList.
  - Example: `GetKeyStore("KeyStore...@af102a")`;
- `AliasInfo(String alias)`
  - List keystore key properties in a JSON object.
  - Example: `AliasInfo('secret')`;

> WIP
