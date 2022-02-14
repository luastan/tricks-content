---
title: Adobe Experience Manager (AEM)
description: Adobe Experience Manager (AEM) is an enterprise-grade CMS and is quite popular among high-profile companies.
badge: Java
---


## Good resources

<iframe class="w-full" height="350" src="https://www.youtube-nocookie.com/embed/EQNBQCQMouk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Writeups

 - [Remote Code Execution (RCE) on Microsoft's 'signout.live.com'](http://www.kernelpicnic.net/2016/07/24/Microsoft-signout.live.com-Remote-Code-Execution-Write-Up.html)

## The tech

Adobe Experience Manager (aka AEM), recently rebranded to Adobe Experience Cloud, is a quite popular enterprise-grace CMS among high-profile companies. 

Under the hood **AEM** is using [Apache Sling](https://sling.apache.org/) wich at the same time is a RESTful web-application framework for [Apache Jackrabbit](https://jackrabbit.apache.org/). On top of that sits the [Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/dispatcher.html?lang=en), wich is intended to pass requests around and is commonly the only protection for AEM sites (Appart from the classic Waf you find in front of anything). This tech stack ends up creating what I personally understand as a huge mess that allows crazy bypassess such as:

 - `/admin` --> 404
 - `/admin.html/.json` --> 200 

This is due to an impropper configuration of the **Dispatcher**, wich is a very common scenario as configuring it is not preciesly a piece of cake:
 - [Adobe's official documentation on Configuring Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration.html?lang=en)


### AEM Common Deploy Models


![AEM Basic and Standard Deployment Model](/aem_deploy_model_basic.jpg)



### Apache Sling & Apache JackRabbit



#### JCR Nodes





### AEM Dispatcher

It actually is a module for the Web server (Apache, IIS...). It provides _security_ (acts more or less like a waf) and _caching layers_. Both are important functionalities to have in mind while pentesting AEM as cache can sometimes fool us away from valid bypasses to this security layer. Two things usually happen here:

 1. In theroy, AEM Dispatcher **offers an extra layer of security** to the AEM infraestructure, but in practice this will, very likely, be the only thing in front of it (**_if any_**). Which is a huge problem as it is relatively straigth-forward to find bypassess for routes for the Dispatcher due to the `.extenxion` mechanics of Apache Sling.
 2. Mantaining every component on _Publish_ updated an securely configured is hard, specially if you think that attackers have no way of accesing them.

With this in mind, the idea is clear, try to find super juicy hidden gems behind the Dispatcher and _profit!_






## Fingerprinting



## Discovery




## Exploitation


### Dispatcher bypasses

This bypasses will help you abuse the other exploitaition techniques exposed here so keep them in mind.

#### Appending stuff (CVE-2016-0957)

Here 

```txt[common_bypasses.txt]
/a.css
/a.html
/a.ico
/%0aa.css
/a.1.json
```

#### Using multiple slashes
Using multiple slashes can get you some bypasses. For example:
 - `///etc.json` instead of `/etc.json`
 - `///bin///querybuilder.json` instead of `///bin///querybuilder.json`


### Default GET Servlets
Huge profit right here; you can stop the `gobuster` you have running in the background already, as this is even going to list even that juicy `/bin/S3cr3t/RANDOM_STR/NO_WAY_THIS_IS_IN_A_WORDLIST` path. This is a feature in Apache Sling you can find more info about in:
 - [Rendering Content - Default GET Servlets](https://sling.apache.org/documentation/bundles/rendering-content-default-get-servlets.html)


So, the idea behind this is that you can retrieve JCR nodes with its props (size, authors/usernames, dates, filenames, child node names...). This can work on any path for the AEM site and you can build _queries_ against them like this:

![How to get JCR nodes](/aem_defaultgetservlet_selection_guide.png)


To _build_ such URLs you can use the following params:

 - **Selectors:**
    - `tidy`: _pretty-prints_ json
    - `infinity`
    - `children`: Holds a list of every children node. You will list literally everything the server has (_Huge profit!_)
    - `childrenlist`: Same behaviour as children(_Huge profit!_)
    - `ext`
    - A numeric value: `-1`, `0`, `1...99999`: The number would typically be the _depth_ of the results (-1 being infinite). A depth of 3 would traverse 3 nodes.
 - **Formats:**
    - `json`
    - `txt`
    - `xml`
    - `res`: Very useful to retrieve files (More info at the [Sling documentation](https://sling.apache.org/documentation/bundles/rendering-content-default-get-servlets.html#streamrendererservlet))

You can use the following script to generate routes to test if the Default Get Servlets are in use:

```python[getservlet_paths.py]
import itertools
import string
import random

CACHE_BUSTER_LEN = 3
JUICY_NODES = ['/', '/etc', '{{ extra-paths /bin }}', '/var', '/apps', '/home']
JUICY_NODES = [p for pair in map(lambda x: (x, x.replace('/', '///')), JUICY_NODES) for p in pair]

GETSERVLET = itertools.product(JUICY_NODES, ('', '.children', '.childrenlist'),('.json', '.1.json', '....4.2.1....json', '.json?{0}.css', '.json?{0}.ico', '.json?{0}.html','.json/{0}.css', '.json/{0}.html', '.json/{0}.png', '.json/{0}.ico','.json;%0a{0}.css', '.json;%0a{0}.png', '.json;%0a{0}.html', '.json;%0a{0}.ico'))

for path in ['{0}{1}{2}'.format(p1, p2, p3.format('{0}')) for p1, p2, p3 in GETSERVLET]:
    print(path.format(''.join(random.choice(string.ascii_letters) for _ in range(CACHE_BUSTER_LEN))))
```

You can execute the script from above as a _"one-liner"_ directly pasting it in `python -c ""` (Nasty but works).



### Query Builder API

It generally sits at `/bin/querybuilder.json`

You can find more information on the following resources:
 - [Adobe - Query Builder API](https://experienceleague.adobe.com/docs/experience-manager-65/developing/platform/query-builder/querybuilder-api.html?lang=en)


It is very useful. Allows you to search and even get node contents. Take 

> TODO


### ReportingServicesProxy SSRF

> TODO


### SalesforceSecretServlet SSRF

> TODO


### SiteCatalystServlet SSRF

> TODO



### AutoProvisioningServlet SSRF

> TODO


### SSRF to RCE

> TODO




## Local testing environment

If you manage to get the required installation `.jar` and license `license.properties` files from Adobe, the following repos might help you out:

 - [AEM & Docker getting started guide](https://github.com/remcorakers/aem-docker-getting-started)
 - [Creating AEM in Docker](https://github.com/PrabhuVignesh/aem-in-docker)