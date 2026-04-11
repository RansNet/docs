# Network Settings

This section covers the network configuration of RansNet SD-WAN branch routers and gateways — interfaces, routing, wireless, redundancy, and connectivity tracking.

All settings described here are accessible under **Device Settings → Network** in the mfusion device management interface, or via the CLI on each device.

---

## Interfaces

Configure the physical and logical network interfaces on the device. Each interface type has its own page:

| Page | Description |
|---|---|
| [Ethernet](iface/ethernet.md) | Layer-3 routed physical ports — IP addressing, MTU, VLAN sub-interfaces, link speed, and other interface settings |
| [VLAN](iface/vlaniface.md) | 802.1Q tagged sub-interfaces for traffic segmentation across a shared physical port |
| [Bridge](iface/bridge.md) | Software Layer-2 bridge combining multiple ports into a single switched segment |
| [WWAN](iface/wwan.md) | Cellular modem interfaces (4G LTE / 5G NR) — APN, band lock, SIM configuration, and 5G mode settings |
| [PPPoE](iface/pppoe.md) | PPPoE client interface for ADSL/VDSL and fibre broadband connections that require PPP authentication |
| [Wi-Fi as WAN](iface/wifiwan.md) | STA (station) mode — connect the built-in Wi-Fi radio to an upstream network as a WAN uplink |

---

## DHCP

[DHCP](dhcp.md) is configured per interface and supports three roles: **DHCP server** (assign addresses to downstream clients), **DHCP client** (obtain an address from an upstream server), and **DHCP relay** (forward DHCP requests to a centralised server on another subnet).

---

## Routing

| Page | Description |
|---|---|
| [Static Routing](route/static.md) | Manually defined routes with optional reachability tracking for conditional route injection and failover |
| [OSPF](route/ospf.md) | Link-state IGP for dynamic route exchange with on-premise infrastructure or between branch routers |
| [BGP](route/bgp.md) | External BGP peering with ISPs or third-party routers — prefix advertisement, path selection, and multi-homing |

---

## Wireless

[Wireless Configuration](wifi.md) covers the built-in Wi-Fi 6 (802.11ax) radios on UA, HSA and UAP series devices. Radios can operate as an **access point** (serving wireless clients), in **EasyMesh** mode (multi-hop self-healing mesh with UAP-520 access points), or in **STA mode** ([Wi-Fi as WAN](iface/wifiwan.md)) to use an upstream Wi-Fi network as a WAN backhaul.

---

## VRRP — High Availability

[VRRP](vrrp.md) provides gateway redundancy by sharing a Virtual IP address (VIP) among a group of routers. The highest-priority member holds the VIP as MASTER; BACKUPs take over automatically on failure. Multiple VRRP groups per interface enable active/active load sharing across VLANs while maintaining full redundancy.

---

## Tracking

[Tracking](tracking.md) is a reachability monitoring engine that probes a target host and triggers a configured action when connectivity is lost. It is used across multiple features to provide intelligent, condition-based behaviour:

| Context | Action on failure |
|---|---|
| Interface | Disable the interface |
| Static route | Withdraw the route from the routing table |
| VRRP | Withdraw from the VRRP group, triggering MASTER handover |
| PBR | Withdraw the policy-based routing rule |
| BGP | Withdraw the BGP prefix advertisement |
| WWAN | Reset the cellular connection or switch from 5G to 4G |
| System | Reboot the router (last-resort watchdog) |
