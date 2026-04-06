# Product Overview

RansNet develops end-to-end SD-WAN/SD-Branch and Wi-Fi Hotspot solutions for enterprise networks and service provider deployments.

![mfusion](../images/start-overview.png)

The product portfolio covers two solution families:

- **SD-WAN / SD-Branch** — branch routers and gateway appliances for secure WAN connectivity, managed through the mfusion orchestrator
- **Wi-Fi Hotspot** — captive portal gateways and enterprise access points for guest Wi-Fi, monetization, and high-density wireless deployments

---

## SD-WAN / SD-Branch Products

### Branch Routers — Series Comparison

Branch routers (mbox) are deployed at sites to provide WAN connectivity, routing, firewall, VPN, and Wi-Fi. Three series are available, differentiated by cellular generation, form factor, and performance tier.

| | **HSA-520** | **UA-520** | **XE-300** |
|---|---|---|---|
| **Cellular** | 4G LTE (Cat 4) | 5G NSA/SA (or 5G RedCap) | 4G LTE or 5G RedCap |
| **Wi-Fi** | Wi-Fi 6 (802.11ax), 2.4/5GHz | Wi-Fi 6 (802.11ax), 2.4/5GHz | Wi-Fi 4 (802.11n), 2.4GHz only |
| **Ethernet** | 5 × GbE (all configurable WAN/LAN) | 5 × GbE (all configurable WAN/LAN) | 1 × FE WAN + 3 × FE LAN (100Mbps) |
| **Firewall throughput** | 1 Gbps | 1 Gbps | 100 Mbps |
| **VPN throughput** | 200 Mbps (IPSec), 300 Mbps (WG) | 200 Mbps (IPSec), 300 Mbps (WG) | 10 Mbps (IPSec), 20 Mbps (WG) |
| **Concurrent devices** | Up to 100 | Up to 100 | Up to 20 |
| **Form factor** | Compact desktop, metal, 195×150×30mm | Compact desktop, metal, 195×150×40mm | Ultra-compact, 115×87×40mm, <200g |
| **Operating temp** | −30°C to 70°C | −30°C to 70°C | −30°C to 70°C |
| **IP rating** | IP30 | IP30 | IP30 |
| **Target use case** | Enterprise branch, retail, SD-Branch with 4G backup | Enterprise branch, SD-Branch with 5G, FWA, IoT | Industrial IoT, harsh environments, compact machine-mount |
| **Datasheet** | [HSA-520](RansNet%20HSA-520%20Datasheet.pdf) | [UA-520](RansNet%20UA-520%20Datasheet.pdf) | [XE-300](RansNet%20XE-300%20V_R_X%20Datasheet.pdf) |

### HSA-520 Model Variants

| Model | Cellular | SIM | Multi-WAN | Notes |
|---|---|---|---|---|
| **HSA-520V** | None | None | Up to 5 WAN | Fixed-line only; no cellular module |
| **HSA-520R** | 4G LTE (Cat 4, Quectel EM05) | Dual (Active/Standby) | Up to 6 WAN | Standard 4G model |
| **HSA-520L2** | 4G LTE × 2 (2× Quectel EM05) | Dual (Active/Active) | Up to 7 WAN | Dual 4G modem for load balancing or separate SIM pools |

### UA-520 Model Variants

| Model | Cellular | 5G Peak Rate | Notes |
|---|---|---|---|
| **UA-520R** | 5G NSA/SA (Qualcomm X62) | DL 3.4 Gbps / UL 550 Mbps | Full 5G Sub-6; primary model for high-bandwidth branch |
| **UA-520X** | 5G RedCap (Qualcomm X35) | DL 223 Mbps / UL 123 Mbps | Cost-efficient 5G; suitable for lower-bandwidth IoT/branch |

### XE-300 Model Variants

| Model | Cellular | WAN | Notes |
|---|---|---|---|
| **XE-300V** | None | Fixed line only | Wired WAN only; lowest cost entry point |
| **XE-300R** | 4G LTE (Quectel EM05) | Fixed + 4G | Standard industrial IoT model |
| **XE-300X** | 5G RedCap (Quectel RG255) | Fixed + 5G | 5G-capable industrial model |

!!! note
    All branch router models include full SD-WAN features: multi-WAN failover, IPSec/SSL/WireGuard/GRE/VXLAN VPN, OSPF/BGP dynamic routing, PBR traffic steering, stateful firewall, VLAN, NAC, SNMP/NetFlow/Syslog, and zero-touch provisioning via mfusion.

---

### SD-WAN Gateway — CMG Series

The CMG series is deployed as the SD-WAN gateway (data center or headquarters side), forming the hub of a hub-and-spoke SD-WAN topology with branch mbox devices as spokes. It is available as a hardware appliance or as a virtual machine (NFV).

| | **CMG-1500** | **CMG-2000** | **CMG-3000** | **CMG-5000** |
|---|---|---|---|---|
| **Tier** | Business | Enterprise | Enterprise | Carrier |
| **Router throughput** | 1.5 Gbps | 10 Gbps | 10 Gbps | 10 Gbps |
| **Firewall throughput** | 1.5 Gbps | 2 Gbps | 5 Gbps | 10 Gbps |
| **Concurrent connections** | 300,000 | 500,000 | 1,000,000 | 2,000,000 |
| **Recommended users** | 200 | 500 | 1,000 | 3,000 |
| **Multi-WAN links** | Up to 3 | Up to 7 | Up to 15 | Up to 28 |
| **LAN/WAN ports** | 4 × GbE | 6 × GbE | 8 × GbE | 8 × GbE |
| **10G expansion** | — | Optional | Optional | Optional |
| **Redundant PSU** | — | — | Optional | Included |
| **Form factor** | Desktop | Desktop | 1U rack | 2U rack |
| **Datasheet** | [CMG Series](RansNet%20CMG%20Product%20Datasheet.pdf) | [CMG Series](RansNet%20CMG%20Product%20Datasheet.pdf) | [CMG Series](RansNet%20CMG%20Product%20Datasheet.pdf) | [CMG Series](RansNet%20CMG%20Product%20Datasheet.pdf) |

All CMG models include: stateful firewall, IPSec/SSL/WireGuard/GRE/VXLAN/SSL VPN, OSPF/BGP, VRRP HA, PBR, QoS, SNMP/NetFlow/Syslog, NFV support, and mfusion zero-touch provisioning.

---

### mfusion — SD-WAN Orchestrator

mfusion is the central management and orchestration platform for all RansNet mbox devices. It can be deployed as a cloud-hosted service or installed on-premise.

Key capabilities:

- **Zero-touch provisioning** — new devices auto-register and receive their configuration on first boot
- **Centralized monitoring** — real-time dashboard, topology map, wireless visibility, NetFlow analytics, and alert management
- **Template-based configuration** — define once, push to hundreds of devices or groups
- **VPN orchestration** — auto-provision hub-and-spoke or full-mesh VPN topologies
- **Multi-tenant** — role-based access with super-tenant dashboard for service providers and MSPs
- **Firmware management** — schedule and deploy firmware updates across the fleet

---

## Wi-Fi Hotspot Products

### Product Comparison

| | **HSG Series** | **UAP-520** | **HSA-520 / UA-520** |
|---|---|---|---|
| **Role** | Captive portal gateway + AP controller | Enterprise access point | Branch router with built-in Wi-Fi |
| **Wi-Fi** | Via any APs | Wi-Fi 6 (AX3000), 2.4/5GHz | Wi-Fi 6 (AX3000), 2.4/5GHz |
| **Captive portal** | Yes — full CMS, templates, ad engine | Via external HSG integration | Via external HSG integration |
| **Authentication** | SMS, email, social, POS, CRM, LDAP, RADIUS, PMS, payment gateway | RADIUS/802.1x, WPA1/2/3-PSK, Web| RADIUS/802.1x, WPA1/2/3-PSK, Web |
| **AP management** | Can be licensed with mfusion features to manage local UAP-520 | Managed by mfusion or standalone or EzMesh Controller | Managed by mfusion or standalone|
| **Target use case** | Hotels, malls, F&B, schools, hospitals — venues needing guest access + monetization | Enterprise / outdoor Wi-Fi coverage extension | Small branch with integrated Wi-Fi |
| **Datasheet** | [HSG Series](RansNet%20HSG%20Series%20Datasheet.pdf) | [UAP-520](RansNet%20UAP-520%20Datasheet.pdf) | See branch router tables above |

### HSG Series — Model Tiers

| Model | Max Throughput | Max Devices | Form Factor |
|---|---|---|---|
| **HSG-200** | 500 Mbps | 200 | Desktop |
| **HSG-400** | 500 Mbps | 400 | Desktop |
| **HSG-800** | 2 Gbps | 800 | Desktop |
| **HSG-1000** | 2 Gbps | 1,000 | Desktop |
| **HSG-2000** | 2 Gbps | 2,000 | 1U rack |
| **HSG-5000** | 2 Gbps | 5,000 | 1U rack |
| **HSG-15000** | 3 Gbps | 15,000 | 2U rack |
| **HSG-25000** | 3 Gbps | 25,000 | 2U rack |

HSG-1000 and above include built-in wireless AP controller (MACC). Redundant PSU available from HSG-2000; included from HSG-15000.

### UAP-520 — Access Point Highlights

The UAP-520 is an enterprise-grade access point (indoor/outdoor, IP67) designed for deployment alongside HSG or mfusion-managed networks.

| Specification | Value |
|---|---|
| **Wi-Fi standard** | 802.11a/b/g/n/ac/ax (Wi-Fi 6, AX3000) |
| **Max throughput** | 2.976 Gbps combined (2.4 + 5 GHz) |
| **Max clients** | 128 (recommended 50) |
| **Ports** | 1 × GbE PoE WAN, 1 × GbE LAN |
| **Power** | PoE 802.3af or DC 12V |
| **Mounting** | Pole / ceiling / wall |
| **IP rating** | IP67 (weatherproof) |
| **Operating temp** | −10°C to 55°C |
| **Datasheet** | [UAP-520](RansNet%20UAP-520%20Datasheet.pdf) |

---

## Which Product Do I Need?

| Scenario | Recommended Product |
|---|---|
| Branch site needing 5G WAN + SD-WAN + Wi-Fi 6 | UA-520R |
| Branch site needing 4G WAN + SD-WAN + Wi-Fi 6 | HSA-520R |
| Industrial/IoT device with 4G backhaul in harsh environment | XE-300R |
| Compact 5G IoT backhaul at lower cost | XE-300X or UA-520X |
| Fixed-line only branch or HQ gateway | CMG-1500 or CMG-2000 |
| High-performance SD-WAN hub / data center gateway | CMG-3000 or CMG-5000 |
| Hotel / mall / venue guest Wi-Fi with captive portal | HSG series (size by device count) |
| Wi-Fi coverage extension for indoor/outdoor enterprise | UAP-520 |
