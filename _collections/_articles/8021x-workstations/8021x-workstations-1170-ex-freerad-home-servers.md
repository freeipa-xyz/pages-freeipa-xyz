---
layout: chapter
chapter_id: 1170
chapter_title: "Extra: Forward to AD NPS"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/d57f643f-8cc3-4957-bf99-050ae97e5982
---


## Home servers

Edit `/etc/freeradius/3.0/sites-available/wifi`.

In our configuration, the site is responsible for proxying logic too.
That is why we configure backend RADIUS inside `wifi` site:

```config
# /etc/freeradius/3.0/sites-available/wifi

### SECTION
### Active Directory RADIUS servers configuration
### They are defined to proxy non-IPA clients to the right RADIUS

home_server ad-nps-1 {
    type = auth
    ipaddr = 172.19.21.4
    port = 1812
    secret = Testing123
}

home_server_pool ad-nps-pool {
  type=fail-over
  home_server = ad-nps-1
  #home_server = ad-nps-2
}

realm ad-nps-realm {
    auth_pool = ad-nps-pool
    nostrip
}

##########


```

