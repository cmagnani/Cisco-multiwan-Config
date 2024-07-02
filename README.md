# Cisco Multi-WAN Configuration

## Purpose  
This document outlines the configuration of my Cisco ISR 1000 (C1111-8P) used in a Dual WAN setup. My ISPs are Free and OVH.

I have two VLANs: one is for the LAN and the other functions as a DMZ.  
One computer in the DMZ can only access the Internet via OVH.

Security zones are enabled but not currently utilized.

> [netconan](https://internetworking.dev/how-to-sanitise-a-cisco-ios-configurations/) was used to anonymize the configuration.

## Interfaces 

### Free provides IP via DHCP

    interface GigabitEthernet0/0/0
     description WAN FREE
     no ip dhcp client request dns-nameserver
     ip address dhcp
     ip nat outside
     zone-member security WAN_OVH
     negotiation auto

> I don't need the Free DNS servers, so I added `no ip dhcp client request dns-nameserver`.

### OVH provides IP via PPPoE

    interface GigabitEthernet0/0/1
     description WAN OVH
     no ip address
     ip mtu 1492
     ip nat outside
     ip tcp adjust-mss 1452
     negotiation auto
     pppoe enable group global
     pppoe-client dial-pool-number 1
     spanning-tree portfast

> PPPoE uses 8 bytes for encapsulation, so the MTU of the interface is 1500-8=1492. Hence, `ip mtu 1492`.

#### Dialer Interface

    interface Dialer1
     mtu 1492
     ip address negotiated
     no ip redirects
     ip nat outside
     zone-member security WAN_OVH
     encapsulation ppp
     ip tcp adjust-mss 1452
     dialer pool 1
     dialer idle-timeout 0
     dialer persistent
     dialer-group 1
     ppp mtu adaptive
     ppp authentication chap callin
     ppp chap hostname netconanRemoved2
     ppp chap password 7 09424B1D1A0A1913053E012724322D3766
     ppp ipcp dns request

## ACL and Route Map
Create a route map for each outside interface. First, create an ACL allowing or denying the inside subnets that can use this interface:

    ip access-list standard 3
     10 permit 192.168.93.0 0.0.0.255
     20 permit 192.168.8.0 0.0.0.255

Then create the route-map:

    route-map WAN-OVH permit 10 
     match ip address 3
     match interface Dialer1

## NAT 
To set up NAT, add:  
- `ip nat inside` to every inside interface  
- `ip nat outside` to every outside interface  

> Be careful with the DHCP interface. `ip nat outside` **must be added after the DHCP client gets the IP address**, otherwise the interface will never get it!

Add `ip nat` in global configuration for each interface with a route map:

    ip nat inside source route-map WAN-FREE interface GigabitEthernet0/0/0 overload
    ip nat inside source route-map WAN-OVH interface Dialer1 overload  

> Since OVH uses PPPoE, add `ip nat` to the Dialer interface.

# TO BE CONTINUED ...