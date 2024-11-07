---
layout: chapter
chapter_id: 1110
chapter_title: "FreeRADIUS: run debug mode"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/f7ad0645-019c-49a7-a411-f5d89c3aa78e
---


## Stop and disable FreeRADIUS service 

Stop and disable FreeRADIUS service while we debugging.

```shell
systemctl stop freeradius
systemctl disable freeradius
```

## Check configuration

Check configuration
```
clear && freeradius -XC
```

## Start FreeRADIUS in debug mode

Start FreeRADIUS in debug mode:
```
clear && freeradius -X
```

Start FreeRADIUS in deep debug mode:
```
clear && freeradius -X -xx
```