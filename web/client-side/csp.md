---
title: Content Security Policy (CSP)
description: Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks.
badge: Web
---

CSP can be really anoying when trying to exploit XSSs. Here are a couple of ways to overcome this limitation

## Abusing `window.name`

> Can be abused on the most restricted CSP environments, but works **only in Chrome**.

Anything stored on the `name` / `window.name` variable will be shared between sites on the same browser tab. This menas that if _attacker-site_ sets the variable with any value, this will be preserved when redirected to a completely different site.

## Abusing `postMessage`

Less of a pain to use than `window.name`, can be used in environments with a not as restrictive CSP. As an example, if `script-src` is restricted, it would be possible to load scripts by passing code through `postMessage`

> Should work on any modern browser if allowed through the website's Content Security Policy
