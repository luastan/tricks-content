---
title: Authentication
badge: server-side
---


## Password-based login


### Brute-force


#### Usernames & passwords

Get some good wordlists


#### Username enumeration

Common vulnerability to enumerate usernames / user IDs based on responses from the server.
The most common differences between responses can be:

- **Status codes**: Unlikely but could actually happen. Specially when trying to get data from other users (for example you get forbidden for valid users and not found for invalid ones).
- **Error messages**: The most common way to enumerate users. Typically found on login forms. The classic _username not found_ vs _wrong password_ or similar.
  - Errors on account locking can also be used to enumerate users. Keep an eye on the behaviour when your account gets locked.
- **Response times**: A website might only check wether the password is correct if the username is valid (hashing and stuff involved), which could make subtle differences in response times. This may be subtle, but **an attacker can make this delay more obvious by entering an excessively long password** that the website takes noticeably longer to handle.


#### Flawed brute-force protection

The server can implement brute-force protection, limiting the ammount of invalid logins an attacker can make for a specific user, an specific IP or some information that can distinguish malicious requests


- **Header bypass**: Every server has a reverse proxy in front nowadays. IP usually reaches final server implementation in the form of a header that you might be able to override. For example using `X-Forwarded-For: `.
- **Counter reset**: Depending on the bussiness logic, valid login attempts might reset brute-force counter. Relogin every X requests can be a way to bypass protection measures.

In some cases, servers might block accounts if certain suspicious criteria is met.
This can be insufficient when the attacker is just traing to gain access to any random account they can.
The following steps might help you approaching this scenario:

1. Establish a list of candidate usernames that are likely to be valid. This could be through username enumeration or simply based on a list of common usernames.
2. Decide on a very small shortlist of passwords that you think at least one user is likely to have. Crucially, the number of passwords you select must not exceed the number of login attempts allowed. For example, if you have worked out that limit is 3 attempts, you need to pick a maximum of 3 password guesses.
3. Using a tool such as Burp Intruder, try each of the selected passwords with each of the candidate usernames. This way, you can attempt to brute-force every account without triggering the account lock. You only need a single user to use one of the three passwords in order to compromise an account.

Account locking also fails to protect against _credential stuffing_ attacks. This involves using a massive dictionary of `username:password` pairs, composed of genuine login credentials stolen in data breaches.


## Vulnerabilities in multi-factor authentication

### Bypassing two-factor authentication

The verfication code is usually asked after the user enters valid credentials.
This means the user is effectively in a "logged in" state.
Check what things can the user do in this state where he has yet to submit the verification code.

You can also try to bruteforce the 2FA code and/or abuse bussiness logic bugs.



## Vulnerabilities in other authentication mechanisms


### Cookie guessing

You can find websites that use a deterministic way to generate session cookies/keys.
Some times it can be the current timestamp hashed with md5, and other times maybe the _"Remember me"_ cookie is just the username in Base64.



### Resetting user passwords and changing user passwords

Pretty much bussiness logic, check information going back and forward between server and client.
Test everything carefully: Look at the different states an account can be, what happens if you put the new password incorrectly but the current is correct.

