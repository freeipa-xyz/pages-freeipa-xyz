---
layout: chapter
chapter_id: 1080
chapter_title: "FreeRADIUS: eap_wifi"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/9d6e064b-dd51-407c-b6e8-467b90bf9594
---


## EAP module for Wi-Fi

Let's define it by editing `/etc/freeradius/3.0/mods-enabled/eap_wifi`.
The EAP module mostry contains TLS and PKI related things.

This includes two main sections: 
* `tls-config tls_wifi_outer` section is where certificates are configured
* `default_eap_type = tls` and section named `tls` is where EAP-TLS method is configured

Please note that I did not issued any certificates yet, so all those certificate files 
do not exist yet.

```
# /etc/freeradius/3.0/mods-enabled/eap_wifi

eap eap_wifi {
  default_eap_type = tls
  timer_expire = 60
  ignore_unknown_eap_types = no
  cisco_accounting_username_bug = no
  max_sessions = ${max_requests}
  tls-config tls_eap_wifi {
    private_key_file = "/etc/freeradius/3.0/tls/private/wifi.key"
    certificate_file = "/etc/freeradius/3.0/tls/certs/wifi.crt"
    ca_path =          "/etc/freeradius/3.0/tls/certs/ca/wifi"
    auto_chain = yes
    cipher_list = "HIGH"
    check_crl = no
    check_all_crl = no
    cipher_server_preference = no
    tls_min_version = "1.2"
    tls_max_version = "1.2"
    ecdh_curve = "prime256v1"
  }

  tls {
    tls = tls_eap_wifi
    configurable_client_cert = no
  }
}
```

