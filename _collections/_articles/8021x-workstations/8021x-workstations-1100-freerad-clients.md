---
layout: chapter
chapter_id: 1100
chapter_title: "FreeRADIUS: configure AP clients"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/fcffe49b-a60e-4cd9-95ac-3312bde31ee9
---



## Configure client (APs) IP addresses and secrets:

### Backup te descriptive file: 

```shell
cp /etc/freeradius/3.0/clients.conf  /etc/freeradius/3.0/clients.conf.orig
```

### Generate **RADIUS shared secret**

I use same secrets for all my access points as it helps decrypting RADIUS protocol if debugging.
I recommend use really random string with at least 16 Upper-Lower-Digits characters.
Adding special chars and increasing length is good if your access points support it easily.
Please avoid using special symbols that may require screening in future: every king of quotes, dollars, comments etc.
You can generate it using this command: 

```shell
LC_ALL=C tr -dc 'A-Za-z0-9' </dev/urandom | head -c 16; echo # 16 chars w\o special symbols
LC_ALL=C tr -dc '[:graph:]' </dev/urandom | head -c 16; echo # 16 chars with special symbols
```

**WARNING**: RADIUS security completely depends on this key.

### Think how you will name your clients

In my network, in future I will have multiple locations, each with multiple access points.
And each location may have it's own VLAN IDs for wireless clients.
That is why I name clients the following way: 
```
wifi@<location-address>@<serial-number-of-device-within-object>

wifi@office-Shanghai-LongtangLane-10@1
wifi@office-Shanghai-LongtangLane-10@2
wifi@office-Shanghai-LongtangLane-10@3
...
wifi@office-Shanghai-DuolunRoad-2@1
wifi@office-Shanghai-DuolunRoad-2@2
...
wifi@office-Singapore-BlairRoad-13k@1
wifi@office-Singapore-BlairRoad-13k@2
...
wifi@warehouse-Singapore-TrengganuRoad-1@1
wifi@warehouse-Singapore-TrengganuRoad-1@2
...

```

### Write config

Edit `/etc/freeradius/3.0/clients.conf`: 
I remove everything from a file and add your Access Point config.

```config
# /etc/freeradius/3.0/clients.conf

client wifi@office-Shanghai-DuolunRoad-2@1 {
        ipaddr = 172.19.21.3/32
        secret = "a92OhH59zq5KF0m2"
    }
client wifi@office-Singapore-BlairRoad-13k@1 {
        ipaddr = 172.19.21.4/32
        secret = "a92OhH59zq5KF0m2"
    }
#...
```


