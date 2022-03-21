---
title: JavaServer Faces (JSF)
description: It is a User Interface (UI) technology for building web UIs with reusable components in java. JSF is mostly used for enterprise applications and a JSF implementation is typically used by a web application that runs on a Java application server like JBoss EAP or WebLogic Server. Now known as Jakarta Server Faces. 
badge: Java
---


## References
 - [Java JSF ViewState (.faces) Deserialization](https://book.hacktricks.xyz/pentesting-web/deserialization/java-jsf-viewstate-.faces-deserialization)
 - [Server-side validation?](https://forum.primefaces.org/viewtopic.php?t=12928)
 - [Dissection Java Server Faces for Penetration Testing](https://secniche.org/jsf/dissecting_jsf_pt_aks_kr.pdf)



## Implementations

JSF defines an interface for the technology. There is multiple implementations and frameworks/libraries that differ on its internals.
JSF defines an interface for the technology. There is multiple implementations and frameworks/libraries that differ on its internals.

### RichFaces


### PrimeFaces
#### References
 - [RCE in Oracle NetBeans Opensource Plugins: PrimeFaces 5.x Expression Language Injection](https://blog.mindedsecurity.com/2016/02/rce-in-oracle-netbeans-opensource.html)
 - [CVE-2017-1000486: Potential EL Injection](https://github.com/primefaces/primefaces/issues/1152)
 - [Security issue - EL injection via DynamicContentStreamer](https://forum.primefaces.org/viewtopic.php?f=3&t=10899)
 - [PrimeFaces And EL Injection Update](https://www.primefaces.org/primefaces-el-injection-update/)



#### EL injection via DynamicContentStreamer
In older versions of PrimeFaces is possible to exploit an Expression Language (EL) injection [^1]:

```
https://{{ target-root localhost:8080/app/faces/main.xhtml }}?primefacesDynamicContent=applicationScope[param.attr]&attr=org.springframework.web.context.WebApplicationContext.ROOT
```

[^1]: https://www.primefaces.org/primefaces-el-injection-update


### Apache MyFaces


### Oracle Mojarra