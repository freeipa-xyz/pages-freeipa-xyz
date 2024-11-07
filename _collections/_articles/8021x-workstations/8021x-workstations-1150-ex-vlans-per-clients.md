---
layout: chapter
chapter_id: 1150
chapter_title: "Extra: Add VLAN IDs per AP"
date:   2024-10-30
theme:  8021x-workstations
permalink: /a/112a1d9f-d207-459a-b7fd-9af67d7ad207
---


## Different VLANs for different locations

I have hundreds of access points distributed across different locations.
The configuration for workstateions remain the same, but some locations require different 
VLAN IDs.

For example, if workstation connects to SSID `ipa8021x` in location `Shanghai`, 
It needs VLAN 101, but if this workstation connects to SSID `ipa8021x` in 
location `Singapore`, it needs VLAN ID 201.
Same for SSID `IPAGuest`: In Shanghai it should be VLAN 102, and in Singapore - VLAN 202.

### Add additional properties to your clients

Clients are [just key-value pairs](https://lists.freeradius.org/pipermail/freeradius-users/2024-October/104920.html) of paramters.
This means we can specify per-location settings passed to clients. That's why we add 
additional keys to clients like `vlan_for_ipa8021x = 1` for each client and later we 
use it in reply. Please note that `ipa8021x` in `vlan_for_ipa8021x` is exact name of SSID.

In `clients.conf` I add additional attributes to clients to define VLAN IDs I want. 
I specify them for two SSIDs: `ipa8021x` and `IPAGuest`. 

Please note that SSIDs are mixed-case, and in config I use the same case

```config
# /etc/freeradius/3.0/clients.conf

client wifi@office-Shanghai-DuolunRoad-2@1 {
        ipaddr = 172.19.21.3/32
        secret = "a92OhH59zq5KF0m2"
        vlan_for_ipa8021x = 101
        vlan_for_IPAGuest = 102
    }
client wifi@office-Singapore-BlairRoad-13k@1 {
        ipaddr = 172.19.21.4/32
        secret = "a92OhH59zq5KF0m2"
        vlan_for_ipa8021x = 201
        vlan_for_IPAGuest = 202
    }
#...
```

### Add additional section to server

In `wifi`\\`authorize` section of server, I will define new rules at just after 
checking `NAS-Port-Type` like this:

```
# /etc/freeradius/3.0/sites-available/wifi

#...
      # I configure Access Points to send be SSID name in Called-Station-Id attribute
      # This attribute's value have form "mac-address:ssid"
      # This built-in filter `rewrite_called_station_id` (defined in 
      #   `policy.d/canonicalization`) normalizes Called-Station-Id and creates 
      #   Called-Station-SSID property in request collection.
      rewrite_called_station_id

      # Check if we have Called-Station-SSID. If none, reject.
      if (!(&Called-Station-SSID)) {
        update request {
          &Module-Failure-Message += 'Rejected: No SSID in Called-Station-Id'
        }
        reject
      }

      # Write Called-Station-Id and Called-Station-SSID to session-state.
      # This should be done because we may lose request data
      update session-state {
        &Called-Station-Id := &request:Called-Station-Id
        &Called-Station-SSID := &request:Called-Station-SSID
      }
    
      # Check if "vlan_for_<ssid>" defined for the client from which request was received.
      # If it is defined, we will save value to session-state as Tunnel-Private-Group-Id
      #   which will be passed in post-auth later to reply. We also define sullimentary
      #   attributes Tunnel-Type and Tunnel-Medium-Type
      # If it is not defined, reject request as we do not know which VLAN to use
      if ("%{client:vlan_for_%{&session-state:Called-Station-SSID}}") { 
        update session-state {
          &Tunnel-Type := VLAN   # rfc3580
          &Tunnel-Medium-Type := IEEE-802   #rfc2628
          &Tunnel-Private-Group-Id := "%{client:vlan_for_%{Called-Station-SSID}}"   # string, rfc2628
        }
      } else {
        update request {
          &Module-Failure-Message += "Rejected: No vlan_for_%{Called-Station-SSID} defined for client %{client:shortname}"
        }
        reject
      }

    # ...
```


