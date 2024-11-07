---
layout: chapter
chapter_id: 1140
chapter_title: "FreeRADIUS: Check against LDAP"
date:   2024-10-30
theme:  8021x-workstations
permalink: /a/e9131bd3-600f-4ae3-8e4f-ea25ffd4c47e
---


## LDAP checks theory

As I noted in very first part of this guide, I made impossible to normally revoke 
certificate issued using `caHostCert` profile.

But I still need mechanism to prevent computer from connecting Wi-Fi network.

In case I become aware of a possible compromise of the certificate and private key of the 
workstation, I should normally block this workstation from authorizing to anything.

FreeIPA, by default, does not have any possibility to disable computer, only to `unenroll`.
Unenrolling may be less secure than deleting computer from directory in some cases, so I
will delete computer from directory in case it is potentially compromised.

The first check that the provided certificate is valid is to check whether the computer 
entity for which the certificate was issued exists in LDAP.

Certificates issued with `caHostCert` profile have Subject Alternative Name (SAN) with 
type `UPN` which looks like `host/hostname.od.freeipa.xyz@OD.FREEIPA.XYZ`.
Within FreeRADIUS, this value is stored in `request:TLS-Client-Cert-Subject-Alt-Name-Upn`
variable in sections `authenticate` (not at the first step) and `post-auth`.
Computers have the save value in LDAP attribute `krbPrincipalName`. 

I will make server to check if there is computer with this UPN in LDAP. 

Secondary, I will check certificate issue date (aka `notBefore`) stored in 
`request:TLS-Client-Cert-Valid-Since` against computer creation date. In FreeIPA, 
computer (and any other object) creation date is stored in `createTimestamp` attribute
which is `operational` attribute. This means this attribute is for internal LDAP server
usage.


## LDAP account for RADIUS service
Create [system account](/t/ipa-system-accounts) for radius service. 

Check if this system account have permissions to read see all necessary attributes 
by searching for `workstation` computer:

```
ldapsearch -x \
  -H 'ldaps://ds1.od.freeipa.xyz:636' \
  -D 'uid=svc_radius_wifi,cn=sysaccounts,cn=etc,dc=od,dc=freeipa,dc=xyz' \
  -b 'cn=computers,cn=accounts,dc=od,dc=freeipa,dc=xyz' \
  -s one \
  -LLL \
  -W \
  '(krbPrincipalName=host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ)' \
  krbPrincipalName createTimestamp
```

As a result you will see something like this:

```
dn: fqdn=workstation.od.freeipa.xyz,cn=computers,cn=accounts,dc=od,dc=freeipa,dc=xyz
krbPrincipalName: host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ
createTimestamp: 20241019002622Z
```

## LDAP filter

We use `request:TLS-Client-Cert-Valid-Since` (string) and 
`request:TLS-Client-Cert-Subject-Alt-Name-Upn` (string) as data for our LDAP request.

Both are strings and typical values are:
```
TLS-Client-Cert-Valid-Since = "241020164306Z"
TLS-Client-Cert-Subject-Alt-Name-Upn = "host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ"
```

From LDAP side corresponding attribute names, typical values, syntax and rules are:
```
createTimestamp = "20241019002622Z"
  Syntax:          GeneralizedTime
  Equality match:  generalizedTimeMatch
  Substring match: none
  Ordering match:  generalizedTimeOrderingMatch

krbPrincipalName = "host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ" 
  Syntax:          IA5String
  Equality match:  caseExactIA5Match
  Substring match: caseExactSubstringsMatch
  Ordering match:  none
```

The `createTimestamp` attribute has ordering mathc `generalizedTimeOrderingMatch`. This
means we can correctly compare it's values with another date value with 
 `<=`, `>=` operators. Note: LDAP does not support strict less than (`<`) and strict 
 greater than (`>`) operators.

Important note: in FreeRADIUS, value of `TLS-Client-Cert-Valid-Since` is written without
the centry information. This is OK for 389DS and it converts year `24` to `2024`. 

In FreeRADIUS [debug output](/a/f7ad0645-019c-49a7-a411-f5d89c3aa78e), 
find lines your client provides in form like this: 

```
...
(7) eap_tls:   TLS-Client-Cert-Valid-Since := "241020164306Z"
(7) eap_tls:   TLS-Client-Cert-Common-Name := "workstation.od.freeipa.xyz"
(7) eap_tls:   TLS-Client-Cert-Subject-Alt-Name-Dns := "workstation.od.freeipa.xyz"
(7) eap_tls:   TLS-Client-Cert-Subject-Alt-Name-Upn := "host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ"
...
```

and query LDAP like this: 
```
ldapsearch -x \
  -H 'ldaps://ds1.od.freeipa.xyz:636' \
  -D 'uid=svc_radius_wifi,cn=sysaccounts,cn=etc,dc=od,dc=freeipa,dc=xyz' \
  -b 'cn=computers,cn=accounts,dc=od,dc=freeipa,dc=xyz' \
  -s one \
  -LLL \
  -W \
  '(&(krbPrincipalName=host/workstation.od.freeipa.xyz@OD.FREEIPA.XYZ)(createTimestamp<=241020164306Z))' \
  krbPrincipalName
```

## LDAP, FreeRADIUS and EAP-TLS

Attributes `TLS-Client-Cert-Valid-Since` and `TLS-Client-Cert-Subject-Alt-Name-Upn` are 
available in `authenticate` (after calling `eap_wifi`) and `post-authenticate` sections, 
but not in `authorize`. This means that we cannot call  `ldap` module in `authorize`. 
But, according to 
[ldap docs](https://networkradius.com/doc/current/raddb/mods-available/ldap.html), 
`ldap` module, when used in `authenticate` section performs authentication by binding to 
ldap with `User-Password` attribute. I do not have user's password, so LDAP is unavailable
in way usually used.

That is why I will use `ldap_wifi.authorize` inside authenticate{Auth-Type eap_wifi{}} block
just after eap_wifi succeeds.

## Create LDAP config

Edit `/etc/freeradius/3.0/mods-available/ldap_wifi`. 

```
# /etc/freeradius/3.0/mods-available/ldap_wifi

ldap ldap_wifi {
  server = 'ldaps://ds1.od.freeipa.xyz:636'
  identity = 'uid=svc_radius_wifi,cn=sysaccounts,cn=etc,dc=od,dc=freeipa,dc=xyz'
  password = 'Ylq4PRFMELRtIz9mPFcG'
  base_dn = 'dc=od,dc=freeipa,dc=xyz' 
  
  update {
    &session-state:Tmp-String-0 := 'createTimestamp'
    &session-state:Tmp-String-1 := 'krbPrincipalName'
  }
  
  user_dn = "${.:instance}-LDAP-UserDn"


  user {
    base_dn = "cn=computers,cn=accounts,${..base_dn}"
    filter = "(&(objectClass=ipahost)(krbPrincipalName=%{request:TLS-Client-Cert-Subject-Alt-Name-Upn})(createTimestamp<=%{request:TLS-Client-Cert-Valid-Since}))"
    scope = 'one'
  }

  options {
    dereference = 'never'
    chase_referrals = no
    rebind = no
    res_timeout = 30
    srv_timelimit = 30
    net_timeout = 3
    idle = 60
    probes = 3
    interval = 3
    ldap_debug = 0x0000
  }
  
  tls {
    start_tls = no
    ca_file = /etc/ipa/ca.crt 
    require_cert    = 'hard'
    tls_min_version = "1.2"
  }
  
  pool {
    start = ${thread[pool].start_servers}
    min = ${thread[pool].min_spare_servers}
    max = ${thread[pool].max_servers}
    spare = ${thread[pool].max_spare_servers}
    uses = 0
    retry_delay = 30
    lifetime = 0
    idle_timeout = 60
  }
}

```


## Add call of ldap_wifi.authorize after EAP authentication succeds:

```
  authenticate {

    Auth-Type eap_wifi {
      eap_wifi {
        fail = reject
        invalid = reject
        reject = reject
      }

      if (&request:TLS-Client-Cert-X509v3-Extended-Key-Usage-OID[*] != "1.3.6.1.5.5.7.3.14") {
        update request {
          &Module-Failure-Message += 'Rejected: No EAPoL EKU'
        }
        reject
      }

      ldap_wifi.authorize {
        noop = reject
        fail = reject
        userlock = reject
        notfound = reject
      }
    }
  }

```

And that's done.






