# Tracking Features

RansNet SD-WAN routers include a built-in tracking engine that continuously monitors network reachability and automatically reacts to changes. Rather than relying on link-state signals alone (which only detect physical failures), tracking probes real connectivity to a target host — so the router can detect upstream failures, degraded paths, or logical connectivity loss even when a link appears physically up.

When a tracked target becomes unreachable, the router takes a configured action: disabling an interface, withdrawing a route, or resetting a cellular connection. When the target recovers, the action is reversed and the original state is restored.

## Use Cases

| Context | What tracking controls |
|---|---|
| Interface settings | Enable or disable the interface |
| Static routing | Inject or withdraw a static route |
| Traffic steering (PBR) | Inject or withdraw a policy-based route |
| BGP network advertisement | Advertise or withdraw a BGP prefix |
| WWAN / SIM | Reset the mobile connection, or switch cellular mode |

## Common Options

All tracking contexts share the same set of core options:

**Tracking method**

- **ICMP** — sends periodic ICMP echo requests (ping) to the target. Supports SLA thresholds (e.g. packet loss rate, latency) for more nuanced failure detection.
- **TCP** — attempts a TCP connection to a specific port on the target (e.g. `tcp/443`). Useful when ICMP is blocked by the upstream network or firewall.

**Source IP** — binds tracking probes to a specific source address. This is important when you need to verify reachability via a particular path, or when the default source IP would be ambiguous (e.g. multi-homed device).

**Log** — when enabled, writes detailed tracking probe results and state transitions to syslog. Use this during commissioning or troubleshooting to understand exactly what the tracker is seeing.

**Reverse** — inverts the tracking behavior. By default, a tracking failure triggers the action (e.g. disable interface, withdraw route). With **Reverse** enabled, the action is triggered when tracking *succeeds* instead. This is useful for scenarios like enabling a backup interface only when the primary becomes unavailable.

---

## Tracking Interfaces

Interface tracking ties the administrative state of an interface to the reachability of an external host. If the tracked host becomes unreachable, the router disables the interface — removing it from routing and freeing the path for a backup. When connectivity is restored, the interface is re-enabled automatically.

A common use case is an optional or backup WAN interface: keep it disabled under normal conditions, and only bring it up when the primary WAN's upstream target is no longer reachable (using the **Reverse** option).

**GUI Example**

Navigate to **Device Settings → Network → Interfaces**. Click an interface to open its settings, then click **Enable Tracking**.

![Tracking](./images/config-track-1.png)

**CLI Example**

```
interface eth0
 ip address dhcp
 track icmp 10.18.18.99 30 src 10.18.18.1 log
```

!!! note
    Always specify a source IP that belongs to a separate, always-up interface — not the interface being tracked. If you use the tracked interface's own IP as the source, probes will always fail once the interface is disabled, preventing it from ever recovering.

---

## Tracking Static Route

Static route tracking allows a route to be conditionally present in the routing table based on reachability. The route is injected when the tracked target is reachable, and withdrawn when it is not.

This pattern is commonly called a **floating static route**. It is typically deployed alongside a backup default route at a higher administrative distance. Under normal conditions, the tracked primary route wins; if the primary path fails, the tracked route is withdrawn and traffic falls back to the backup automatically.

**GUI Example**

Navigate to **Device Settings → Network → Static Routing**. Click to add or edit a route.

![Tracking](./images/config-track-2.png)

In this example, the default route via `100.100.100.1` is only active when the nexthop is reachable. If the upstream fails, the route is withdrawn and a backup (higher-metric) default takes over.

!!! tip
    For deeper upstream validation, track an IP further into the upstream network rather than just the immediate nexthop. This catches scenarios where the nexthop itself is up but connectivity beyond it has failed.

**CLI Example**

```
ip route 0.0.0.0/0 nexthop 100.100.100.1 track icmp 100.100.100.1 30
```

---

## Tracking Policy-Based Route

Policy-based routing (PBR) allows traffic to be steered based on criteria beyond the destination address (e.g. source subnet, DSCP mark). PBR tracking works the same way as static route tracking — the PBR rule is only active when the tracked target is reachable.

The key advantage of PBR over static routing in a multi-WAN scenario is precedence. In most deployments, the WAN interfaces use DHCP, which installs kernel-level default routes. Kernel routes supersede static routes, so a static backup default may never be reached. PBR operates above the kernel routing table, so a tracked PBR rule will take effect regardless of what kernel or static routes are present — giving you reliable, deterministic traffic steering even in mixed-WAN environments.

**GUI Example**

Navigate to **Device Settings → SD-WAN → Traffic Steering**. Click to add or edit a rule.

![Tracking](./images/config-track-3.png)

**CLI Example**

```
ip pbr policy 100 src 192.168.8.0/22
ip pbr 100 nexthop 100.100.100.1 track icmp 1.1.1.1 15
```

In this example, traffic from `192.168.8.0/22` is steered via `100.100.100.1` as long as `1.1.1.1` is reachable. If tracking fails, the PBR rule is withdrawn and traffic follows normal routing.

---

## Tracking BGP Route

By default, BGP advertises a connected prefix only when the corresponding interface is up. This works well for physical interfaces, but causes a problem on platforms where LAN interfaces are always logically up regardless of physical connectivity.

On HSA-520 series devices, the LAN interface is a VLAN interface — it remains administratively up even when the downstream switch or cable is unplugged. BGP has no visibility into this failure and will continue advertising the LAN prefix to peers, attracting traffic that cannot actually be delivered. BGP tracking solves this by tying prefix advertisement to a tracked host on the LAN segment.

When the tracked LAN host becomes unreachable (indicating a switch failure, cable pull, or downstream device outage), the router withdraws the BGP advertisement immediately, preventing upstream peers from forwarding traffic down a broken path.

**GUI Example**

Navigate to the BGP configuration section (under **SD-WAN** or **Dynamic Routing**, depending on your deployment), and open the advertised network entry:

![Tracking](./images/config-track-4.png)

**CLI Example**

```
router 65051
 ...
 network 192.168.5.254/24 track icmp 192.168.5.6 30
```

Set the tracked IP to a known-static host on the LAN (e.g. a server or a managed switch with a fixed IP). When that host becomes unreachable, the prefix is withdrawn from BGP within one tracking interval.

---

## Tracking WWAN/SIM Connections

WWAN tracking has one additional tracking method beyond ICMP and TCP: **RSRQ** (Reference Signal Received Quality). RSRQ reflects radio signal quality rather than upper-layer IP reachability, making it useful for detecting degraded radio conditions before connectivity is fully lost.

### Connection Reset Behavior

WWAN tracking behaves differently from interface tracking. Rather than permanently disabling the interface on failure, the system briefly resets the cellular connection and immediately attempts to re-establish it. This is intentional.

Cellular networks are not perfectly stable — as a device moves between coverage areas, it can remain radio-associated with a distant tower while losing routable connectivity through that tower's backhaul. From the router's perspective, the interface is "up" but traffic goes nowhere. A connection reset forces the modem to re-register, pick up a nearby tower, and re-establish a fresh PDP context, restoring connectivity within seconds.

This behavior also benefits devices using **multi-profile SIMs** — a single SIM card with multiple carrier profiles loaded. When one profile fails, the reset cycle allows the system to try the next available profile automatically, without manual intervention.

**GUI Example**

Navigate to **Device Settings → Network → WWAN**.

![Tracking](./images/config-track-5.png)

**CLI Example**

```
interface wwan0
 track icmp 1.1.1.1 30
```

### Switch Cellular Mode (5G → 4G Failover)

RansNet cellular routers can automatically fail over between 5G and 4G when the preferred mode loses end-to-end connectivity.

This addresses a specific failure mode: the 5G radio and the RAN (Radio Access Network) appear operational, but the 5G core network (5GC) has a fault. The modem shows a 5G connection, but no traffic gets through. Simply resetting the connection won't help — the modem will re-register on the same 5G core. The solution is to switch the radio access technology entirely, forcing the router onto LTE, which uses a separate 4G core network (EPC) and a different backhaul path.

```
interface wwan0
 nr-mode NR5G track icmp 1.1.1.1 30 failover LTE
```

In this configuration, the router operates in NR5G mode by default. If `1.1.1.1` becomes unreachable (indicating a 5G core failure), it switches to LTE mode.
