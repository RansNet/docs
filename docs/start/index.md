# Getting Started

Welcome to the RansNet documentation site. This guide covers the full lifecycle of deploying, configuring, and managing RansNet SD-WAN and Wi-Fi solutions — from unboxing a device through to advanced network configuration and API integration.

---

## Who is RansNet?

RansNet is a Singapore-based technology vendor who develops end-to-end SD-WAN and wireless solutions built around two core components:

- **mbox** — the physical or virtual network appliance deployed at each site. Available in several series (UA, HSA, CMG, XE) with varying cellular, Wi-Fi, and routing capabilities. See [Product Overview](overview.md) for more details.
- **mfusion** — the cloud or on-premise SD-WAN orchestrator. Provides zero-touch provisioning, centralized monitoring, configuration management, and policy orchestration across all deployed mbox devices.

A typical deployment has one or more mbox devices at branch sites, reporting to a single mfusion instance that gives your network team a unified view and control plane.

---

## How to Use This Documentation

The documentation is organized by function. Use the sections below as a map to find what you need:

| Section | What it covers | Start here if... |
|---|---|---|
| **Getting Started** *(this section)* | Product overview, bootstrapping, onboarding, hardening | You are deploying a device for the first time |
| **[Device Monitoring](../monitor/index.md)** | Dashboard, topology, alerts, per-device drill-down | You want visibility into device health and network status |
| **[Network Settings](../config/index.md)** | Interfaces, VLANs, DHCP, routing, Wi-Fi, tracking | You are configuring network interfaces and connectivity |
| **[SD-WAN](../sdwan/index.md)** | VPN, multi-WAN, traffic steering, QoS | You are setting up WAN connectivity or SD-WAN policies |
| **[Security & Firewall](../security/index.md)** | Firewall rules, DNAT/SNAT, DNS filtering, RADIUS | You are hardening or securing the network |
| **[Sample Scenarios](../scenarios/wanfailover.md)** | WAN failover, L2/VRF over SD-WAN, Wi-Fi over SD-WAN | You want end-to-end deployment examples |
| **[API Documentation](../api/netflow.md)** | NetFlow, monitoring, hotspot, syslog APIs | You are integrating mfusion with external systems |

---

## New Device Deployment Workflow

If you are setting up a RansNet device for the first time, follow this sequence:

1. **[Product Overview](overview.md)** — Identify your hardware series and understand how mbox and mfusion work together.
2. **[Bootstrapping](device/bootstrap.md)** — Connect to the device via console or SSH and configure the initial mfusion connection so the device can call home.
3. **[Provisioning](device/provision.md)** — Apply a configuration template through mfusion for zero-touch setup.
4. **[Onboarding](device/onboard.md)** — Register and verify the device in mfusion, confirm it appears in the dashboard.
5. **[Hardening](device/harden.md)** — Apply security best practices before putting the device into production.

---

## Support

| Channel | Details |
|---|---|
| Email support | [support@ransnet.com](mailto:support@ransnet.com) |
| Website | [www.ransnet.com](https://www.ransnet.com) |
