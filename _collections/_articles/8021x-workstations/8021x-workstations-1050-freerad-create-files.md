---
layout: chapter
chapter_id: 1050
chapter_title: "FreeRADIUS: create files"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/79351526-1aac-4cba-83e5-0ba657f0faaa
---


## Configure FreeRADIUS

### Create necessary files and remove unnecessary

After you install freeradius, there will be many mods and several sites enabled. We disable all sites and many mods: 

```shell
rm /etc/freeradius/3.0/mods-enabled/eap
rm /etc/freeradius/3.0/mods-enabled/chap
rm /etc/freeradius/3.0/mods-enabled/digest
rm /etc/freeradius/3.0/mods-enabled/mschap
rm /etc/freeradius/3.0/mods-enabled/ntlm_auth
rm /etc/freeradius/3.0/mods-enabled/pap
rm /etc/freeradius/3.0/mods-enabled/unix
rm /etc/freeradius/3.0/mods-enabled/passwd
rm /etc/freeradius/3.0/mods-enabled/files
rm /etc/freeradius/3.0/mods-enabled/soh
rm /etc/freeradius/3.0/mods-enabled/dynamic_clients
rm /etc/freeradius/3.0/mods-enabled/echo
rm /etc/freeradius/3.0/mods-enabled/expiration
rm /etc/freeradius/3.0/mods-enabled/logintime
rm /etc/freeradius/3.0/mods-enabled/linelog
rm /etc/freeradius/3.0/mods-enabled/exec
rm /etc/freeradius/3.0/mods-enabled/detail
rm /etc/freeradius/3.0/mods-enabled/detail.log
rm /etc/freeradius/3.0/mods-enabled/preprocess
rm /etc/freeradius/3.0/mods-enabled/radutmp
rm /etc/freeradius/3.0/mods-enabled/sradutmp
rm /etc/freeradius/3.0/mods-enabled/realm
rm /etc/freeradius/3.0/mods-enabled/replicate
rm /etc/freeradius/3.0/mods-enabled/unpack
rm /etc/freeradius/3.0/mods-enabled/utf8
rm /etc/freeradius/3.0/sites-enabled/*
```

Check if freeradius is failing to start because there is nothing to listen
```
freeradius -XC
```

If get final message like this, then it is OK
```
The server is not configured to listen on any ports.  Cannot start
```


Next, create 2 sites and 2 eap modules:

```shell
touch /etc/freeradius/3.0/mods-available/eap_wifi
touch /etc/freeradius/3.0/mods-available/ldap_wifi
touch /etc/freeradius/3.0/sites-available/wifi

ln --symbolic --target-directory=/etc/freeradius/3.0/mods-enabled  ../mods-available/eap_wifi
ln --symbolic --target-directory=/etc/freeradius/3.0/mods-enabled  ../mods-available/ldap_wifi
ln --symbolic --target-directory=/etc/freeradius/3.0/sites-enabled ../sites-available/wifi

chown -R freerad:freerad /etc/freeradius/3.0/sites-*
chown -R freerad:freerad /etc/freeradius/3.0/mods-*

chmod 640 /etc/freeradius/3.0/sites-available/*
chmod 640 /etc/freeradius/3.0/mods-available/*

```

