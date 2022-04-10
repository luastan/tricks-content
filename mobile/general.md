---
title: General tricks
position: 3
---


## Bring your own CA

Mobile devices are usually very picky when it comes to certificates HTTPS. In some cases the configuration used by default on burp's certificate does not fullfill some requirements.

```bash
wget http://web.mit.edu/crypto/openssl.cnf -O openssl.cnf
openssl req -x509 -days 365 -nodes -newkey rsa:2048
```
