---
layout: chapter
chapter_id: 1000
chapter_title: "Create system account"
date:   2024-10-31
theme:  ipa-system-accounts
permalink: /a/3a29e627-6ff0-4ef4-9604-7c3a4c0af1cb
---

There is no built-in tool for creating `system accounts` in IPA.

There is a third-party tool [noahbliss/freeipa-sam](https://github.com/noahbliss/freeipa-sam), 
but I will show you how to make a system account without it.

To create `system account`, you need to make an LDAP *modify* query.
I will show you how to create an account for FreeRADIUS that processes Wi-Fi requests.

First, I make account name. I recommend to name it starting `svc_` (for `service`).
Note that I use **service name** which is RADIUS, and **not software name** which is FreeRADIUS.
For IMAP service, which is running `dovecot`, I use well-known in mail community name `mda`
which stands for Mail Delivery Agent, and I do not use `dovecot` in service account name.
So for FreeRADIUS wifi it will be `svc_radius_wifi`.

Next, generate a plaintext password for this account.
I use [this article](/t/password-generation-bash) to generate complex password.

Expiration time `20380119031407Z` currently means *never expires*

Prepare the query.

```
dn: uid=svc_radius_wifi,cn=sysaccounts,cn=etc,dc=od,dc=freeipa,dc=xyz
changetype: add
objectclass: account
objectclass: simplesecurityobject
uid: svc_radius_wifi
userPassword: Ylq4PRFMELRtIz9mPFcG
passwordExpirationTime: 20380119031407Z
nsIdleTimeout: 0

```

Run on DS server:

```
ldapmodify -x -H ldap://127.0.0.1 -D 'CN=Directory Manager' -W
```

Authenticate. Then paste your query and press `ENTER` until you see `adding new entry ...` message.
Press `CRTL + D` to leave ldapmodify tool.