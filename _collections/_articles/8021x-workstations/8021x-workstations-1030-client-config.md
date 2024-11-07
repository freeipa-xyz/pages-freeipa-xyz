---
layout: chapter
chapter_id: 1030
chapter_title: "Configure client"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/87dd6cec-f296-41f5-b488-8bdbb73b2aab
---


## Issue workstation certificate

Use `certmonger` which is installing with `freeipa-client` to obtain a certificate for host. Secure certificate key with permissions for `root` user only.

Create folders for storing certificate: 
```shell
mkdir --parents --mode=700 /etc/pki/tls/private
mkdir --parents --mode=755 /etc/pki/tls/certs
```

Tell to `certmonger` to request certificate from CA. Use certificate profile `ca802_1xCert` on CA, and set local name for certificate tracking to `802_1x`: 

```shell
ipa-getcert request \
  --id=802_1x \
  --profile=ca802_1xCert \
  --renew \
  --keyfile=/etc/pki/tls/private/802_1x.key \
  --key-owner=root \
  --key-perms=600 \
  --certfile=/etc/pki/tls/certs/802_1x.crt \
  --cert-owner=root \
  --cert-perms=644 \
  --ca-file /etc/pki/tls/certs/802_1x.ca.crt \
  --wait \
  --wait-timeout=60 \
  --key-size=2048 
```

Check if certificate is obtained: 
```shell
ipa-getcert list 
# ...
# eku: id-kp.14
```

## Configure Network Manager

I will human-readable name for NM connection. As an example, name it by SSID name. 
My SSID name is `ipa8021x`, so I will name it `wifi_ipa8021x`. 
That value goes to `connection.id` option.

I will also make recognizable connection UUID. I want It to be same on all computers in my network.
I will generate random UUID and replace first part with *hex-word* and the last part is zeroed.
I went to [UUID Generator](https://www.uuidgenerator.net/version4) and generated random UUID, then replaced parts of it: 

```
5436899d-20a0-49da-8cd5-897f779714e2 # Generated this UUID

decade9d-20a0-49da-8cd5-897f779714e2 # replaced first six letters with "decade" to make
^^^^^^                               #   this ID more recognizable

decade99-20a0-49da-8cd5-897f779714e2 # filled first block with "99" to make it even more
      ^^                             #   recognizable

decade99-20a0-49da-8cd5-897f77970000 # replaced last 4 letters with 0s so I can increase
                                ^^^^ #   it by one if something changes

decade99-20a0-49da-8cd5-897f779a0000 # I don't like 0s not separated from other digits
                               ^     #   so replaced pre-0s digit 7 with letter a
```

The value `decade99-20a0-49da-8cd5-897f779a0000` goes to `connection.uuid` parameter.

Next, I will add additional validation of domain suffix match: we trust only our RADIUS
servers, so I will add my domain name here: `od.freeipa.xyz`. Since NM 1.24 it's possible
to specify multiple values here throug `;`, but I have only one.

So, we can now create a command that creates or replaces our connection:

```
nmcli connection delete id wifi_ipa8021x & \
nmcli connection add \
  type wifi \
  save yes \
  -- \
  connection.id "wifi_ipa8021x" \
  connection.uuid "decade99-20a0-49da-8cd5-897f779a0000" \
  connection.type "802-11-wireless" \
  connection.autoconnect "yes" \
  connection.autoconnect-priority "50" \
  connection.zone "work" \
  connection.metered "no" \
  connection.lldp "disable" \
  connection.mdns "no" \
  connection.llmnr "no" \
  connection.dns-over-tls "no" \
  connection.mptcp-flags "disabled" \
  ipv4.method "auto" \
  ipv4.may-fail "no" \
  ipv4.link-local "disabled" \
  ipv6.method "disabled" \
  802-11-wireless.ssid "ipa8021x" \
  802-11-wireless.mode "infrastructure" \
  802-11-wireless.wake-on-wlan "ignore" \
  802-11-wireless-security.key-mgmt "wpa-eap" \
  802-11-wireless-security.proto "wpa" \
  802-11-wireless-security.wps-method "disabled" \
  802-1x.optional "no" \
  802-1x.eap "tls" \
  802-1x.anonymous-identity "anonymous-od-type-a" \
  802-1x.system-ca-certs "off" \
  802-1x.auth-timeout "30" \
  802-1x.ca-cert-password-flags "not-required" \
  802-1x.client-cert-password-flags "not-required" \
  802-1x.private-key-password-flags "not-required" \
  802-1x.password-raw-flags "not-required" \
  802-1x.pin-flags "not-required" \
  802-1x.domain-suffix-match "od.freeipa.xyz" \
  802-1x.ca-cert "file:///etc/pki/tls/certs/802_1x.ca.crt" \
  802-1x.client-cert "file:///etc/pki/tls/certs/802_1x.crt" \
  802-1x.private-key "file:///etc/pki/tls/private/802_1x.key"

```


