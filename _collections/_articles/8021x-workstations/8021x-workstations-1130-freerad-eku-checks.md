---
layout: chapter
chapter_id: 1130
chapter_title: "FreeRADIUS: Check certificate EKUs"
date:   2024-10-30
theme:  8021x-workstations
permalink: /a/17d8f1bd-6067-45ec-9766-89b4791bfdf8
---


## Add EKU check

Previously, we defined special Extended Key Usage (EKU) in `caHostCert` certificate 
profile called `EAP-over-LAN` (OID `1.3.6.1.5.5.7.3.14`). 

I will check if client certificate authenticating with EAP has this EKU in certificate:

In file `/etc/freeradius/3.0/sites-available/wifi`, I extend `eap_wifi` with 
`Auth-Type eap_wifi` block `authenticate` section:

```
# /etc/freeradius/3.0/sites-available/wifi

#...
  authenticate {
    
    Auth-Type eap_wifi {
      eap_wifi {
        fail = return
        invalid = return
        reject = return
      }
      
      if (&request:TLS-Client-Cert-X509v3-Extended-Key-Usage-OID[*] != "1.3.6.1.5.5.7.3.14") {
        update request {
          &Module-Failure-Message += 'Rejected: No EAPoL EKU'
        }
        reject
      }
    }
  }
#...
```
