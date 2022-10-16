---
title: Leaks
badge: Google Cloud
---


With an API key you can make that company pay for your use of the API, but you won't be able to escalate privileges.

## Useful regexes

```shell
TARGET_DIR="{{ target-dir app_directory }}"

# Service account keys
grep -HPzr "(?s){[^{}]*?service_account[^{}]*?private_key.*?}" "$TARGET_DIR"

# Legacy GCP creds
grep -HPzr "(?s){[^{}]*?client_id[^{}]*?client_secret.*?}" "$TARGET_DIR"

# Google API keys
grep -HPnr "AIza[a-zA-Z0-9\\-_]{35}" "$TARGET_DIR"

# Google OAuth tokens
grep -HPnr "ya29\.[a-zA-Z0-9_-]{100,200}" "$TARGET_DIR"

# Generic SSH keys
grep -HPzr "(?s)-----BEGIN[ A-Z]*?PRIVATE KEY[a-zA-Z0-9/\+=\n-]*?END[ A-Z]*?PRIVATE KEY-----" "$TARGET_DIR"

# Signed storage URLs
grep -HPinr "storage.googleapis.com.*?Goog-Signature=[a-f0-9]+" "$TARGET_DIR"

# Signed policy documents in HTML
grep -HPzr '(?s)<form action.*?googleapis.com.*?name="signature" value=".*?">' "$TARGET_DIR"

# Legacy FCM server key
grep -HPinr 'AAAA[A-Za-z0-9_-]{7}:[A-Za-z0-9_-]{140}' "$TARGET_DIR"
```

## Finding API keys

The [keytest](https://github.com/luastan/keytest) tool can help you find and check the scope of API keys. It is worth checking it out.

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
