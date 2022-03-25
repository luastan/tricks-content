---
title: SSL / TLS configuration
badge: Web
description: Analyse the server SSL / TLS strength in terms of algorithms and configuration
---

This is usually pretty straight forward. There are some pretty good tools that will do all the job for you. My personal favourite is [testssl.sh](#testssl.sh)

## Testssl.sh

Very popular and nice tool. It is also very esasy to use with docker which is pretty cool. You can find more info on the [testssl.sh repo](https://github.com/drwetter/testssl.sh). The default branch with the latest stable changes currently is (_for some reason_) the <code><smart-variable variable="testssl-branch">3.1dev</smart-variable></code>.

### Installation

The tool only needs `OpenSSL` or `LibreSSL`, as well as `/bin/bash` with the standard scripting tools like `awk` or `sed`. With that you can just clone the repo and run the script. I personally like to have it in a docker image that I build myself, but you can just use the image the author pushes to dockerhub (`drwetter/testssl.sh`). This would be the commands to _"install"_ the tool:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker build -t {{ image-name testssl }} "https://github.com/drwetter/testssl.sh.git#{{ testssl-branch 3.1dev }}"
```

</template>
<template v-slot:cli>

```bash
git clone --depth 1 "https://github.com/drwetter/testssl.sh.git" {{ testssl-dir /mnt/d/pentesting/tools/testssl }}
```

</template>
</smart-tabs>

> It might come installed or on the repos from Kali, Parrot or similar distros. I've seen it having problems or being outdated on such scenarios, so I just recommend to clone the repo and leave those skid commodities behind.

### Usage

The usage is really easy. It just takes the target domain as an argument:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker run --rm -it {{ image-name testssl }} {{ target-domain example.com }}
```

</template>
<template v-slot:cli>

```bash
{{ testssl-dir /mnt/d/pentesting/tools/testssl }}/testssl.sh {{ target-domain example.com }}
```

</template>
</smart-tabs>
