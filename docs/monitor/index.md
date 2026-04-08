# Device Monitoring

mfusion provides end-to-end monitoring of all managed RansNet devices — SD-WAN gateways, access points, hotspot gateways, and other network appliances — from a single centralised interface. The monitoring stack covers real-time device health, performance trending, wireless RF visibility, traffic analysis, and automated alerting, giving network administrators full operational awareness across distributed fleets.

Navigate to **ORCHESTRATOR → Monitoring** to access all monitoring features. Use the **[Entity]** button in the top-right corner to scope the view to a specific customer or site.

---

## Monitoring Capabilities

### Real-Time Device Health

mfusion continuously polls all managed devices using an agent-based collection model. Each device pushes metrics — uptime, CPU, memory, interface status, WAN IP, firmware version, and cellular signal — to mfusion at configurable intervals. Thresholds are evaluated in real time against trigger conditions; when a threshold is breached, an alert is raised and email notifications are dispatched automatically to all users with access to the affected entity.

### Performance Trending

Historical metrics are retained per device, enabling administrators to plot performance graphs over time. Trending data informs capacity planning decisions — for example, identifying WAN links that are consistently saturated at peak hours, or devices that are gradually exhausting memory under load.

### Wireless RF Visibility

For deployments using RansNet UAP access points, mfusion collects per-AP and per-client RF metrics: signal strength (RSSI), noise floor, SNR, channel utilization, and connection rates. This provides early detection of interference, channel congestion, or clients with degraded signal that would otherwise be invisible at the network layer.

### NetFlow Traffic Analysis

The vessel HSG and supported gateways export NetFlow records to mfusion, providing granular per-connection traffic visibility: source and destination IP, port, protocol, bytes transferred, and session duration. This supports bandwidth accounting, security investigation, and dispute resolution without requiring packet capture.

### Automated Alerting

Alert triggers are pre-configured per device type through monitoring templates (`Template_HSA`, `Template_mbox`). When a trigger condition is met — device unreachable, memory threshold exceeded, link down — mfusion creates an alert entry and sends email notifications. Alerts are severity-classified (Disaster, High, Average, Warning, Information) for prioritised response.

---

## In This Section

| Topic | Description |
|---|---|
| [Dashboard](dashboard.md) | Customisable overview of device health and performance metrics across the selected entity |
| [Devices](hosts.md) | Per-device monitoring view with status, uptime, WAN IP, and drill-down into items, triggers, and historical graphs |
| [Topology](topo.md) | Visual network map editor with live device and link status; includes auto-generated SD-WAN topology |
| [Wireless](wifi.md) | Real-time RF visibility into access points, SSIDs, and associated client signal quality |
| [Alerts](alerts.md) | Centralised alert log with severity levels, timestamps, and email notification management |
| [NetFlow](netflow.md) | Granular per-connection traffic records for bandwidth analysis, security investigation, and dispute resolution |
| [Settings](settings.md) | Advanced monitoring configuration — templates, items, triggers, and bulk update tools (super-admin) |

---

## How Monitoring Is Structured

mfusion organises monitoring around **entities** — a hierarchical grouping of devices that maps to your customer or site structure. All monitoring views are entity-scoped: selecting an entity at the top of any monitoring page filters the data to devices within that entity only.

Each device is assigned a **monitoring template** at provisioning time. Templates define the complete set of metrics, alert thresholds, and graphs for that device type. Administrators can inspect and override individual items or triggers at the per-device level without modifying the underlying template, or use the mass update tools in Settings to apply changes across many devices at once.
