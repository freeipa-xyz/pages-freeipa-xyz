---
layout: chapter
chapter_id: 1090
chapter_title: "FreeRADIUS: issue server certificates"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/5e6c1ad7-8be3-40c4-85cd-976645c25f15
---



## Issue server certificates

Prepare folders for FreeRADIUS certificates: 
```shell
mkdir --parents --mode=750 /etc/freeradius/3.0/tls/private
mkdir --parents --mode=755 /etc/freeradius/3.0/tls/certs/ca/wifi
chown --recursive freerad:freerad /etc/freeradius/3.0/tls
```

Issue a certificate adding it to certmonger:
```shell
ipa-getcert request \
  --id=radius_wifi \
  --profile=caIPAserviceCert \
  --renew \
  --keyfile=/etc/freeradius/3.0/tls/private/wifi.key \
  --key-owner=freerad \
  --key-perms=640 \
  --certfile=/etc/freeradius/3.0/tls/certs/wifi.crt \
  --cert-owner=freerad \
  --cert-perms=644 \
  --ca-file=/etc/freeradius/3.0/tls/certs/ca/wifi/wifi.ca.crt \
  --wait \
  --wait-timeout=60 \
  --key-size=2048 
```

Fix permissions:
```
chown --recursive freerad:freerad /etc/freeradius/3.0/tls
```

Check if cert is issued:
```shell
ipa-getcert list --id=radius_wifi
```

