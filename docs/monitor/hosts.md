# Devices

## What This Does
The Devices page provides a centralized view of all monitored devices within the selected entity, including their status, performance, and key system information.

## Why It Matters
It allows administrators to quickly assess device health, identify issues, and perform detailed analysis from a single interface.

## Where to Access
Navigate to **ORCHESTRATOR → Monitoring → Devices**, then select the desired entity using the **[Entity]** button in the top-right corner.

---

## Overview

The **Hosts** tab displays a summary of all enabled devices within the selected entity.

Key information includes:

- Device model  
- Firmware version  
- Device status  
- Uptime  
- Public and private WAN IP addresses  

![mfusion host dashboard customization](../images/monitor-hosts-1.png)

---

## Customize Hosts Table

The hosts summary table can be customized to display relevant information.

Click **Customize columns** to:

- Select which fields to display  
- Adjust the table view based on your operational needs  

Changes will take effect after saving.

![mfusion host dashboard customization](../images/monitor-hosts-2.png)

---

## Drill-in Analysis

To view detailed information for a specific device:

1. Click the device hostname  
2. Select **Host Dashboard**  

![mfusion host dashboard customization](../images/monitor-hosts-4.png)

---

## Host Information Fields

| Field | Description |
|-------|-------------|
| **Host / Alias** | Name used to identify the device. Configurable under **ADMIN → Hosts** |
| **Entity** | The entity to which the device belongs |
| **Model** | Hardware model (HSG, CMG, HSA, UA), automatically detected by mfusion |
| **Status** | Current device state; can also be used to enable or suspend the device |
| **Device Uptime** | Time elapsed since the last reboot |
| **Firmware Version** | Current firmware version running on the device |
| **WAN Traffic Inbound** | Latest inbound traffic rate on the WAN interface |
| **WAN Traffic Outbound** | Latest outbound traffic rate on the WAN interface |
| **WAN IP (Public)** | External WAN IP address as seen from the internet |
| **WAN IP (Private)** | Internal WAN IP address from the device perspective |

---

## Host Dashboard

Within the Host Dashboard, you can navigate through the following tabs:

- **Summary** — Overall device health and associated links  
- **Items** — Monitoring items (enabled and disabled)  
- **Graphs** — Historical performance data  
- **Alerts** — Device-specific alert history  

Use the date/time selector (top-right corner) to adjust the data range.

---

### Host Summary

The Host Dashboard layout is customizable.

Click the **[Customize]** button in the top-right corner to:

- Select widgets  
- Configure displayed sections  

![mfusion host dashboard customization](../images/monitor-hosts-5.png)

---

### Monitoring Items

The **Items** tab displays all monitoring items, including disabled ones (listed at the bottom).

Monitoring items are defined by templates assigned during provisioning (e.g. `Template_HSA`, `Template_mbox`).

You can override item status per device by:

- Clicking the action button next to each item  
- Enabling or disabling as required  

![mfusion host dashboard customization](../images/monitor-hosts-6.png)

---

### Monitoring Triggers

Triggers are defined in monitoring templates and can be customized per device. When the monitoring items values match the defined trigger threshold, alerts are created and email to all users within the entity.

You can:

- Enable or disable specific triggers  
- Override default template behavior  

![mfusion host dashboard customization](../images/monitor-hosts-7.png)

---