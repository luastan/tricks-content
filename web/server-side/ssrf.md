---
title: Server-Side Request Forgery (SSRF)
description: Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing.


badge: Web
---


In a typical SSRF attack, the attacker might cause the server to make a connection to internal-only services within the organization's infrastructure. In other cases, they may be able to force the server to connect to arbitrary external systems, potentially leaking sensitive data such as authorization credentials.


## Common SSRF attacks

### SSRF attacks against the server itself


### SSRF attacks against other back-end systems


## Circumventing common SSRF defenses


### DNS Rebinding

As explained in this bug bounty report. DNS Rebinding can be used to abuse a race condition on the IP whitelist.



### Bypass filtesr via open redirection



## Additional Tips & Trickz

### Blind SSRF vulnerabilities
