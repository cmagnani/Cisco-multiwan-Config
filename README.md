# Cisco Multi-WAN Configuration

## Dump of my Cisco C1111-8P 

> [netconan](https://internetworking.dev/how-to-sanitise-a-cisco-ios-configurations/) used to anonymize the config  

I have 2 ISPs, Free and OVH.  
Free provides IP via DHCP, and OVH provides IP via PPPOE.  
  
I have 2 VLANs: one is LAN and the other one is like a DMZ.  
One computer from the DMZ can only reach the Internet via OVH.

### NAT 
To set up NAT, just add:  
- [ ] `ip nat inside` to every inside interface  
- [ ] `ip nat outside` to every outside interface  

> Just be careful with the DHCP interface. `ip nat outside` **must be added after the DHCP client gets the IP address**, otherwise the interface will never get it!!  

Add `ip nat` for each interface with a route map:

    ip nat inside source route-map WAN-FREE interface GigabitEthernet0/0/0 overload
    ip nat inside source route-map WAN-OVH interface Dialer1 overload  

### Route Map
Create a route map for every interface, create an ACL allowing or denying the inside network that should use this interface:

    ip access-list standard 3
     10 permit 192.168.93.0 0.0.0.255
     20 permit 192.168.8.0 0.0.0.255

Then create the route-map:

    route-map WAN-OVH permit 10 
     match ip address 3
     match interface Dialer1

# TO BE CONTINUED ...
