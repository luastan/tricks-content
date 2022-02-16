---
title: Checklist
badge: Web
position: 9
description: A simple checklist to remember the tests you should make on a web assessment
---

This is a test for chacklist persistance on the browser localStorage

* [ ] Session handling
* [ ] Test tokens for meaning
* [ ] Test tokens for predictability
* [ ] Insecure transmission of tokens
* [ ] Disclosure of tokens in logs
* [ ] Mapping of tokens to sessions
* [ ] Session termination
* [ ] Session fixation
* [ ] CSRF
* [ ] Cookie scope
* [ ] Decode Cookie \(Base64, hex, URL etc.\)
* [ ] Cookie expiration time
* [ ] Check HTTPOnly and Secure flags
* [ ] Use same cookie from a different effective IP address or system
* [ ] Access controls
* [ ] Effectiveness of controls using multiple accounts
* [ ] Insecure access control methods \(request parameters, Referer header, etc\)
* [ ] Check for concurrent login through different machine/IP
* [ ] Bypass Anti-CSRF tokens
* [ ] Weak generated security questions
* [ ] Path traversal on cookies
* [ ] Reuse cookie after session closed
* [ ] Logout and click browser "go back" function \(Alt + Left arrow\)
* [ ] 2 instances open, 1st change or reset password, refresh 2nd instance
* [ ] With privileged user perform privileged actions, try to repeat with unprivileged user cookie.
