---
layout: chapter
chapter_id: 1000
chapter_title: "Theory"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/a380fdcd-92a5-4d6c-b3c7-ada8dd2b6d7f
---

## Theory

The goal is to add ability for FreeIPA-joined workstations to be able to connect to our Wi-Fi network safe way.
To both secure, simplify and automate this task, this is used:
* Authenticated subjects are workstations, not users.
* Workstations request their individual certificates themselves.
* Certificates are issued by FreeIPA-integrated DogTag PKI.
* Workstations renew certificates automatically using `certmonger`
* Certificates are short-living (90 days in my case)

The authentication I will use is `EAP-TLS`, which is quite common authenticateion mechanism.

I also have Active Directory joined clients connecting to the same Access Point. 
I need to proxy their authentication to Microsoft Network Policy Server (NPS) RADIUS server.

#### Important note about certificates in this guide

Despite the fact that I use certificate authentication, I will make impossible to use 
certificate revocation mechanisms provided by FreeIPA for certificates I use for Wi-Fi 
authentication. The reason is simple: I do not want to overload my LDAP with certificates, 
that is why I will tell IPA not to save certificates to `userCertificate` attribute.

The consequences of such a decision will be as follows:
* Certificates are not saved to LDAP `userCertificate`
* As FreeIPA does not have information about certificate, it cannot revoke it.
* Checking certificate revocation by FreeRADIUS is useless, as certificates will never be revoked

Compensatory measures that I will apply:
* Certificates will be short-lived: 60 days.
* FreeRADIUS will check for existence of identity in LDAP
* FreeRADIUS will will check if `creationDate` of LDAP object is less than certificate's `notBefore` date
* * For those who never seen `creationDate` attribute: this is **operational** LDAP attribute and may not be displayed by default.

Please note that if you plan using this certificate for anything else, you should understand it is not revokeable.


