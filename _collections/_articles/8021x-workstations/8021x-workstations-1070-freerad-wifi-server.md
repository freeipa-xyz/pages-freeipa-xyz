---
layout: chapter
chapter_id: 1070
chapter_title: "FreeRADIUS: wifi server"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/a7a67328-a9cb-4efb-be96-72f6cac4ec07
---


## Wi-Fi server

Edit `/etc/freeradius/3.0/sites-available/wifi`.

Now I define the site that is responsible for communicating with client

```config
# /etc/freeradius/3.0/sites-available/wifi
server wifi {

  listen {
    type=auth    # Listen only auth requests.
    ipv4addr = * # Listen on all IPv4 addresses
    port = 1812  # Listen on port 1812 (default for RADIUS access requests)
    proto = udp  # Listen UDP (default for RADIUS)
  }

  authorize {       # `authorize` section is also named `pre-authenticate`. 
                    #   This is where requests starts processing

    # Check request for user-name existence. If not, reject request.
    #   EAP connections may have 2 fields for this: `username` and 
    #   `anonymous identity` or `phase 1 identity`. If no value is provided on client for
    #   `anonymous identity`, then same identity as phase 2 identity is used, and this
    #   behavoir compromises username.
    #   IPA workstations will use string `anonymous-od-type-a` here.
    #   I will check if identity starts with `anonymous-od-`. If true, this client should
    #   be authenticated with this FreeRADIUS. Else, it should be proxied to AD NPS.
    #   Note: FreeRADIUS checks if phase 1 identity starts with `anonymous`. If not, 
    #   FreeRADIUS will log warning about identity may be compromised.
    if (!(&User-Name)) {
      update request {
        &Module-Failure-Message += 'Rejected: User-Name not provided.'
      }
      reject
    }

    # Check if user-name starts with `anonymous-od-`
    # For nowm we will reject request.
    # `i` means case-insensitive
    if (&User-Name !~ /^anonymous-od-/i) {
      update request {
        &Module-Failure-Message += 'Rejected: by identity'
      }
      reject
    }

    # This server processes only Wi-Fi authentication, so I check if request 
    #   has NAS-Port-Type attribute and it's value is Wireless-802.11. 
    #   If not, I reject RADIUS request.
    if (!(&NAS-Port-Type && &NAS-Port-Type == Wireless-802.11)) {
      update request {
        &Module-Failure-Message += 'Rejected: NAS-Port-Type is not Wireless-802.11.'
      }
      reject
    }

    # If we still here, call eap processing
    # If EAP started, this module will return "handled".
    # If EAP still is in progress, this module will return "ok" or "updated".
    # This means EAP is not ready yet and we can not authenticate client yet.
    # This module is placed at very beginning of this section, because it stops
    # processing (`return`s) section until all EAP exchange completes.
    # If you put some logic before EAP module, this logic will be executed every
    # EAP message.
    eap_wifi {
      ok = return
      updated = return
      handled = return
    }

    # We should reach this code only when EAP fails
    update request {
      &Module-Failure-Message += 'Rejected: EAP failed'
    }
    reject
  }

  authenticate {
    eap_wifi
  }

  post-auth {
    if (session-state:User-Name && reply:User-Name && request:User-Name && (reply:User-Name == request:User-Name)) {
        update reply {
                &User-Name !* ANY
        }
    }

    update {
        &reply: += &session-state:
    }

    remove_reply_message_if_eap

    Post-Auth-Type REJECT {
        attr_filter.access_reject

        # Insert EAP-Failure message if the request was
        # rejected by policy instead of because of an
        # authentication failure
        eap_wifi

        #  Remove reply message if the response contains an EAP-Message
        remove_reply_message_if_eap
    }

    if (EAP-Key-Name && &reply:EAP-Session-Id) {
        update reply {
                &EAP-Key-Name := &reply:EAP-Session-Id
        }
    }
  }

  pre-proxy {
  }

  post-proxy {
  }
}

```

As you noted, we use `eap_wifi` module here. I will describe in next part.


