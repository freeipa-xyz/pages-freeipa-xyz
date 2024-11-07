---
layout: chapter
chapter_id: 1000
chapter_title: "Possible solutions"
date:   2024-10-31
theme:  ipa-dns-no-forwarding
permalink: /a/3e88a7b4-f280-48be-b359-5ad44eda6a0b
---

## Problem explained:

FreeIPA DS DNS server does not resolves names outside zones hosted by FreeIPA.
But records from zones managed by FreeIPA are resolved.

```

[root@ds2 ~]# nslookup -type=A example.com. 127.0.0.1
Server:         127.0.0.1
Address:        127.0.0.1#53

** server can't find google.com: SERVFAIL



[root@ds2 ~]# nslookup ds1.od.freeipa.xyz 127.0.0.1
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   ds1.od.freeipa.xyz
Address: 172.19.21.21
```

## Possible reason:

This is probably related to DNS frowarder settings became wrong.

In FreeIPA, DNS forwarders are configured through LDAP. 

There are two types of places where it is configured: 
* Global forwarders
* Per-Server forwarders

Per-Server forwarders have higher priority over global forwarders.

### Check DNS is actually working

First of all, check if requests to forwarder from problematic FreeIPA DS server are working:

```
[root@ds2 ~]# nslookup google.com. <ip-address-of-your-forwarder>
```

If this fails, you possibly have network problems. 
If this succeeds, let's check that forwarder you used is actually configured as forwarder 
on problematic FreeIPA DS server

### Global forwarders

To check if your domain has global forwarders configured, check `ipa dnsconfig-show`
```
[root@ds2 ~]# ipa dnsconfig-show
  Global forwarders: 172.19.21.254, 172.19.21.253
  IPA DNS servers: ds1.od.freeipa.xyz, ds2.od.freeipa.xyz
```

If there are no `Global forwarders` record, then they are not configured. 
I'd suggest setting them later after you fix your current issue.

If there are wrong forwarders, set correct with command:
```
ipa dnsconfig-mod --forwarder=172.19.21.254 --forwarder=172.19.21.253
```

Just note: this command sets `idnsForwarders` attribute of 
`cn=dns,<dc=your-domain,dc=tld>`. 
This is multi-valued attribute and each value should contain single IP address of DNS forwarder.

### Per-Server forwarders

Per-server forwarders are stored in LDAP as multi-valued `idnsForwarders` attribute of 
`idnsserverid=<dns-server-name.your-domain.tld>,cn=servers,cn=dns,<dc=your-domain,dc=tld>`.
AFAIK, there is no CLI tool to modify it. 

Connect to LDAP as `CN=Directory Manager` and check records above. 
```
ldapsearch -x -D 'CN=Directory Manager' -b 'cn=servers,cn=dns,dc=od,dc=freeipa,dc=xyz' -H 'ldap://127.0.0.1' -W -LLL -s one idnsForwarders 
```

You will see something like that:
```
ldapsearch -x -D 'CN=Directory Manager' -b 'cn=servers,cn=dns,dc=od,dc=freeipa,dc=xyz' -H 'ldap://127.0.0.1' -W idnsForwarders -LLL -s one
Enter LDAP Password: 
dn: idnsserverid=ds1.od.freeipa.xyz,cn=servers,cn=dns,dc=od,dc=freeipa,dc=xyz
idnsForwarders: 172.19.21.254

dn: idnsserverid=ds2.od.freeipa.xyz,cn=servers,cn=dns,dc=od,dc=freeipa,dc=xyz
idnsForwarders: 172.19.21.1
```

If there are incorrect servers listed in `idnsForwarders` attribute of problematic server, modify them:
Prepare modify request in text editor. Get `dn` of problematic server from previous command. 
Note lines with `-` symbols: they are part of request, even last one.


```
dn: idnsserverid=ds1.od.freeipa.xyz,cn=servers,cn=dns,dc=od,dc=freeipa,dc=xyz
changetype: modify
delete: idnsForwarders
-
add: idnsForwarders
idnsForwarders: 172.19.21.201
-
add: idnsForwarders
idnsForwarders: 172.19.21.202
-
```

Run `ldapmodify -x -D 'CN=Directory Manager' -H 'ldap://127.0.0.1' -W` and authenticate.

Then paste your query, press enter 1 or 2 times until you see `modifying entry "idnsserverid=...`

Press `CTRL + D` to exit `ldapmodify`.

You're done

### Other possible problems

Of course there may be many other problems with DNS not listed here: for example, your forwarder is not accessible

