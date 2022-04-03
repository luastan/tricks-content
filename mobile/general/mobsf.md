---
title: MobSF
description: Mobile Security Framework (MobSF) is an automated, all-in-one mobile application (Android/iOS/Windows) pen-testing, malware analysis and security assessment framework capable of performing static and dynamic analysis.
badge: mobile
---

## Installation

MobSF is basically a web application running on Python.
I like to use it with docker.

<smart-tabs variable="tool-docker-vs-linux-vs-windows" :tabs="{'docker': 'Docker', 'cli': 'Linux / Bash', 'windows': 'Windows'}">
<template v-slot:docker>

```bash
docker build -t {{ image-tag mobsf }} "https://github.com/MobSF/Mobile-Security-Framework-MobSF.git#master"
```

Additionally, if you would like to safely store past scan results and change the configuration, create a duirectory to mount the data and change the owner to `9901:9901`[^1] :

[^1]: The owner needs to be set to uid and gid  as stated by the [official documentation](https://mobsf.github.io/docs/#/docker).

```bash
mkdir -p {{ persistance-dir /mnt/d/pentesting/docker/mobsf }}
chown 9901:9901 {{ persistance-dir /mnt/d/pentesting/docker/mobsf }}
```

</template>
<template v-slot:cli>

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
./setup.sh
```

</template>
<template v-slot:windows>

```batch
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
setup.bat
```

</template>
</smart-tabs>

## Usage

<smart-tabs variable="tool-docker-vs-linux-vs-windows" :tabs="{'docker': 'Docker', 'cli': 'Linux / Bash', 'windows': 'Windows'}">
<template v-slot:docker>

For persistance, mount your desired directory as a volume to `/home/mobsf/.MobSF`:

```bash
docker run --rm -it -p "{{ lhost 127.0.0.1 }}:{{ lport 8080 }}" -v "{{ persistance -dir /mnt/d/pentesting/docker/mobsf }}:/home/mobsf/.MobSF" {{ image-tag mobsf }}
```

If you **do not want** persistance, use the following command:

```bash
docker run --rm -it -p {{ lhost 127.0.0.1 }}:{{ lport 8080 }} {{ image-tag mobsf }}
```

</template>
<template v-slot:cli>

```bash
./run.sh {{ lhost 127.0.0.1 }}:{{ lport 8080 }}
```

</template>
<template v-slot:windows>

```batch
run.bat {{ lhost 127.0.0.1 }}:{{ lport 8080 }}
```

</template>
</smart-tabs>

### Static analyzer

> TODO

### Dynamic Analyzer

Didn't test the dynamic analyzer for malware analysis but for your classic security assessment, this is not really usefull.

### API

> Maybe some scripting can automate the process
