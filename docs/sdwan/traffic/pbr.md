# Traffic Steering (Policy-Based Routing)

Explain what's PBR, how it works, what it does, when do we need to use it etc.

By default, a router (or layer-3 gateway) routes packets based on packet destination address header. It looks up its default routing table (statically configured or learnt through dynamic routing protocols eg. OSPF/BGP), chooses the most specific route (default route will be used if no matching route), and selects nexthop/gateway address. 

But in certain complex scenarios, we need to forward packets based on source, or destination or applications, or a combination of multiple criterion. That's where we need traffic steering (a.k.a, Policy-Based Routing/PBR). PBR can be very useful when interconnecting several private networks, or sharing multiple upstream ISP links, or directing traffic for special purposes (eg. redirect to external proxies, firewalls or caching engines etc).

RansNet routers support PBR based on either (or combination) of below:

Source address

Destination address

Applications (protocol, port number, FQDN, object-groups)

CONFIG NOTES

- PBR happens at inbound interface when packets enters the interface, so "ip pbr policy..." should always be matching packets when they enter the interface (therefore use firewall-set inbound to mark packets)
- If you set nexthop as interface, make sure it has a default route (with a default gateway), the system will auto learn the default gateway as nexthop IP. 
- like firewall rules, PBR rules are evaluated sequentially from top down (lower rule numbder to higher)
- When we configure PBR on HSG (with hotspot service running), we need to take note of a few things:

  - we must use "By Packet Flow" match the local VLAN/network. Don't use "ip pbr policy xx src y.y.y." to match by source IP address. Because the packets will not match this rule due to the order of operations between hotspot and PBR processes
  - when configuring "By Packet Flow" matching, because hotspot service generates dynamic inbound tunnel interface so we are not sure which tunnel no. to use, just use tun+ as the inbound interface, together with src IP/Network to narrow to a particular vlan/network.

please also study current CLISH CLI and script to highlight other features or pre-cautions needed during deployments.

CONFIGURATION EXAMPLE - Based on source (or destination)



In this example, we are trying to achieve below objectives:

clients from 172.16.30.0/24 will go out from ISP1 link for Internet access

clients from 172.16.40.0/24 will go out from ISP2 link for Internet access



CONFIGURATION EXAMPLE - Based on Applications

In this example, we are trying to achieve below objectives:

HTTP (TCP/80) access will go out from ISP1 link

HTTPS (TCP/443 and UDP/443) access will go out from ISP2 link

