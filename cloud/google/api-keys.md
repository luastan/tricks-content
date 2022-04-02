---
title: Api Keys
badge: Google Cloud
description: An API key is a simple encrypted string that identifies an application without any principal. They are useful for accessing public data anonymously, and are used to associate API requests with your project for quota and billing.
---


With an API key you can make that company pay for your use of the API, but you won't be able to escalate privileges.

## Finding API keys

The [keytest](https://github.com/luastan/keytest) tool can help you find and check the scope of API keys. It is worth checking it out

## Exploits

### FCM server keys

You always want to test Google API keys against the Firebase Cloud Messaging (FCM) service. As it is possible that the key is allowed to send notifications to every user of the legitimate application. There's 3 possible FCM server key types:

- A FCM server key of the regex: `AAAA[A-Za-z0-9_-]{7}:[A-Za-z0-9_-]{140}`
- A Legacy FCM server Key of the regex : `AIzaSy[0-9A-Za-z_-]{33}`. This gets automatically added to your GCP project.
- A GCP key with FCM privileges of the same regEx as above (`AIzaSy[0-9A-Za-z_-]{33}`). In This variation, the key is created under a GCP project and then provided the essential privileges.

The following request can help you detect such environments:

```shell
curl --header "Authorization: key={{ api-key AIzaSyZ-... }}" \
     --header Content-Type:"application/json" \
     https://fcm.googleapis.com/fcm/send \
     -d "{\"registration_ids\":[\"TEST\"]}"
```

If the API key is vulnerable, the server will respond with **Status 200**.
