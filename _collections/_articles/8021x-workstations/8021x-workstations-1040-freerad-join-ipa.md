---
layout: chapter
chapter_id: 1040
chapter_title: "FreeRADIUS: Join FreeIPA"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/02f2fba2-b05b-42d4-8437-7f17c33e96a1
---


## Join FreeRADIUS server to FreeIPA domain
This is done to automatically issue FreeRADIUS certificates like this

```shell
ipa-client-install \
  --hostname "$( hostname --short ).od.freeipa.xyz" \
  --domain "od.freeipa.xyz"  \
  --ntp-pool time.freeipa.xyz \
  --no-ssh \
  --no-sshd \
  --no-sudo \
  --ssh-trust-dns \
  --enable-dns-updates \
  --password=7Qexxxxxxxxxx 
```
