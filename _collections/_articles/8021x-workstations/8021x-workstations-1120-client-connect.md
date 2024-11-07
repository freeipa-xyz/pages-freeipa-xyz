---
layout: chapter
chapter_id: 1120
chapter_title: "Connect client"
date:   2024-10-30
theme:  8021x-workstations
permalink: /a/2063bcf2-bfda-466c-856c-23217e9500b6
---



## On client, run

On client, run:
```
nmcli connection down id wifi_ipa8021x & nmcli connection up id wifi_ipa8021x
```

Check FreeRADIUS logs.

Client should connect successfully. Stop (`CTRL+C`) FreeRADIUS debugging. 
Disconnect client using `nmcli connection down id wifi_ipa8021x`.

