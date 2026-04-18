# Multi-WAN (MWAN)

Multi-WAN (MWAN) provides outbound traffic load balancing and automatic failover across multiple WAN links. It is included as a standard feature on HSG, CMG, and HSA series devices with no additional licensing required.

MWAN aggregates available bandwidth across multiple ISP connections, monitors each link continuously, and automatically redirects traffic to surviving links when a connection drops. It also supports per-flow traffic steering rules — equivalent to policy-based routing (PBR) — with built-in failover.

---

## How It Works

- **Load balancing** — distributes outbound sessions across WAN links according to configurable weights. Weights are relative: a weight of `2` carries twice the traffic of a weight of `1`.
- **Failover** — each WAN link is monitored via repeated ping tests to its default gateway. If a link fails, MWAN automatically redirects traffic to the remaining active links.
- **Traffic steering** — rules match outbound traffic by source IP, destination IP, destination port, or protocol, and direct matching flows to a designated WAN group with failover support.
- **Unlimited WAN links** — the practical limit is the number of available physical interfaces on the device.

---

## Important Notes

!!! note "Per-Connection Load Balancing"
    MWAN balances on a per-IP-connection basis. A single-stream speed test or FTP transfer to one server will only use one WAN link at a time and will not reflect the aggregate bandwidth. Load balancing benefits are realised when multiple hosts access multiple destinations simultaneously, spreading sessions across links.

!!! tip "Metric and Weight"
    - **Same metric** across interfaces → load balancing. Traffic is distributed in proportion to the configured weights.
    - **Different metrics** → active/standby failover. Lower metric = preferred; higher metric = standby, activated only when all lower-metric links fail.
    - Weights are only compared between interfaces sharing the same metric value.

!!! note "Persistent Balancing"
    MWAN supports a **persistent** mode where sessions from the same source IP reuse the same WAN link for the duration of a configurable timeout. This is required for applications that enforce source IP consistency — for example, some HTTPS sites with inline IPS/HIPS that flag source IP changes as suspicious activity. Use persistent rules sparingly: each persistent session entry consumes system resources, which becomes significant in large networks.

!!! note "DNS Resolution with ISP Restrictions"
    Some ISPs block external DNS queries transiting their network. If users lose Internet access when traffic is balanced to a particular ISP link, verify DNS resolution is functional on that link. In this case, configure an internal DNS server or use the ISP-provided DNS server address instead of public resolvers such as `8.8.8.8`.

---

## Configuration Example

The following example uses three ISP links — two active (load balanced) and one standby (failover only):

| Interface | ISP | Subnet | Capacity | Metric | Weight | Role |
|---|---|---|---|---|---|---|
| `eth0` | ISP1 | `172.16.1.0/24` | 10 Mbps | 1 | 1 | Active — load balanced |
| `eth1` | ISP2 | `172.16.2.0/24` | 20 Mbps | 1 | 2 | Active — load balanced |
| `eth2` | ISP3 | `172.16.3.0/24` | 30 Mbps | 2 | 3 | Standby — failover only |
| `eth3` | LAN  | `172.16.99.0/24` | — | — | — | Office LAN |

`eth0` and `eth1` share metric `1` and load-balance in a 1:2 ratio, proportional to their 10/20 Mbps capacities. `eth2` has metric `2`, so it remains on standby and only becomes active if both `eth0` and `eth1` fail. The weight of `eth2` has no effect on the `eth0`/`eth1` balancing — it would only apply if another interface also shared metric `2`.

![Three-ISP MWAN topology](./images/mwan-1.png)

---

## GUI Configuration

Navigate to **Device Settings → SD-WAN → Multi-WAN**.

### Prerequisites

Before configuring MWAN, ensure each WAN interface is fully configured with its IP address and default route, and verify connectivity by pinging the ISP default gateway on each link. Refer to the Ethernet Interface section for interface configuration details.

### Step 1: Add MWAN Interfaces

Click **Add Interface** to register each WAN interface with MWAN.

![MWAN interface list and Add Interface form](./images/mwan-2.png)

| Field | Description |
|---|---|
| **WAN Interface** | The physical WAN interface to register (e.g. `eth0`, `eth1`, `ppp0`) |
| **Enable** | Toggle to activate this MWAN interface entry |
| **Metric** | Failover priority. Lower value = preferred. Interfaces with the same metric are load-balanced against each other |
| **Weight** | Relative traffic share among interfaces with the same metric. A weight of `2` receives twice the sessions of a weight of `1` |
| **Tracking** | Enable link health monitoring via ICMP ping |
| **Track Host** | IP address to ping for health checks — typically the ISP default gateway |
| **Interval (s)** | Ping interval in seconds (default: `5`) |
| **Attempt** | Number of consecutive failed pings before the link is declared down (default: `5`) |

Click **Continue** after each entry. Repeat for all WAN interfaces.

### Step 2: Define Balancing Policies

Click **Add Rule** to specify which traffic uses which WAN group. Rules are evaluated top-down; the first matching rule applies.

![Balancing Policy rule list](./images/mwan-3.png)

| Field | Description |
|---|---|
| **Protocol** | Traffic protocol to match: `IP` (all), `TCP`, `UDP`, or `ICMP` |
| **Source (Port)** | Source IP/subnet and optional port. Use `any` to match all sources |
| **Destination (Port)** | Destination IP/subnet and optional port |
| **Persistent** | When enabled, sessions from the same source IP reuse the same WAN link for the session lifetime |

Click **Save** when all rules are defined.

!!! tip
    For most deployments, a single default rule matching all destinations (`0.0.0.0/0`) is sufficient. If persistence is required for specific applications (e.g. HTTPS), add a dedicated persistent rule for that traffic **above** the default catch-all rule.

### Step 3: Configure Firewall Rules

Navigate to **Device Settings → Security → Firewall Policies**.

Add one **Access Rule** and one **SNAT Rule** per WAN interface to permit outbound traffic and translate internal source addresses to the WAN public IP.

![Firewall Access and SNAT rules for MWAN](./images/mwan-4.png)

| Rule Type | Operation | Direction |
|---|---|---|
| **Access Rule** | `Permit` | `Outbound (ethX)` — one rule per WAN interface |
| **SNAT Rule** | `Overload` | `Outbound (ethX)` — one rule per WAN interface |

---

## CLI Configuration

### Static WAN IP

Three ISP links with static addresses. `eth0` and `eth1` load-balance at metric `1`; `eth2` is standby at metric `2`:

```
hostname CMG-MWAN
!
interface eth0
 description "to ISP1 Internet"
 enable
 ip address 172.16.1.2/24
 mwan-group 99
  track 172.16.1.1
  metric 1
  weight 1
!
interface eth1
 description "to ISP2 Internet"
 enable
 ip address 172.16.2.2/24
 mwan-group 99
  track 172.16.2.1
  metric 1
  weight 2
!
interface eth2
 description "to ISP3 Internet"
 enable
 ip address 172.16.3.2/24
 mwan-group 99
  track 172.16.3.1
  metric 2
  weight 3
!
interface eth3
 description "to LAN"
 enable
 ip address 172.16.99.1/24
!
ip route 0.0.0.0/0 nexthop 172.16.1.1
ip route 0.0.0.0/0 nexthop 172.16.2.1
ip route 0.0.0.0/0 nexthop 172.16.3.1
!
ip dhcp-server 172.16.99.0 255.255.255.0
 description "DHCP for LAN users"
 dns 8.8.8.8 8.8.4.4
 router 172.16.99.1
 domain ransnet.com
 range 172.16.99.5 172.16.99.254
 static epson-printer 64:EB:8C:F9:30:C4 172.16.99.2
 start
!
firewall-access 100 permit outbound eth0
firewall-access 101 permit outbound eth1
firewall-access 102 permit outbound eth2
!
firewall-snat 100 overload outbound eth0
firewall-snat 101 overload outbound eth1
firewall-snat 102 overload outbound eth2
!
mwan-rule 100 tcp dport 443 group 99 persistent remark "https traffic"
mwan-rule 101 dst 0.0.0.0/0 group 99 remark "default rule"
!
```

Key points:

- All three interfaces share `mwan-group 99`. Metric and weight values determine balancing behaviour within the group.
- A static default route (`ip route 0.0.0.0/0 nexthop <gateway>`) is required for each static WAN link.
- MWAN rules are evaluated top-down. The persistent HTTPS rule (`100`) must appear before the default catch-all rule (`101`).

### Dynamic (DHCP) WAN IP

When WAN interfaces use DHCP, the device learns default gateways automatically — no static default routes are needed:

```
hostname MWAN
!
interface eth0
 description "to ISP1"
 enable
 ip address dhcp
 mwan-group 0
  track 172.16.1.1
  timer 3 3
  metric 1
  weight 10
!
interface eth1
 description "to ISP2"
 enable
 ip address dhcp
 mwan-group 0
  track 172.16.2.1
  timer 3 3
  metric 1
  weight 20
!
interface eth2
 description "to LAN"
 enable
 ip address 172.16.3.1/24
!
mwan-rule 11 tcp dport 443 group 0 persistent remark "https traffic"
mwan-rule 14 dst 0.0.0.0/0 group 0 remark "default rule"
!
firewall-access 10 permit outbound eth0
firewall-access 11 permit outbound eth1
!
firewall-snat 10 overload outbound eth0
firewall-snat 11 overload outbound eth1
!
```

!!! note "Mixed Static and DHCP"
    When one WAN link uses a static IP and another uses DHCP, explicit default routes must be added for **both** links — including the DHCP link — even though the DHCP link would otherwise learn its gateway automatically. MWAN requires all participating routes to be explicitly present in the routing table:

    ```
    ip route 0.0.0.0/0 nexthop 138.75.64.1
    ip route 0.0.0.0/0 nexthop 3g-lte0
    ```

### PPPoE WAN

When a WAN link uses PPPoE, configure `mwan-group` under the virtual `ppp0` interface rather than the physical Ethernet uplink:

```
hostname mbox
!
interface eth0
 description "to ISP1 (static)"
 enable
 ip address 172.21.2.88/24
 mwan-group 0
  track 172.21.2.1
  metric 1
  weight 2
!
interface eth1
 description "to ISP2 (PPPoE uplink)"
 enable
 pppoe 11111 22222
!
interface eth2
 description "to LAN"
 enable
 ip address 192.168.10.1/24
 dhcp-server
  dns 8.8.8.8 8.8.4.4
  range 192.168.10.5 192.168.10.254
!
interface ppp0
 mwan-group 0
  track 182.253.32.1
  metric 1
  weight 1
!
ip route 0.0.0.0/0 nexthop 172.21.2.1
ip route 0.0.0.0/0 nexthop ppp0
!
mwan-rule 11 tcp dport 443 group 0 persistent remark "https traffic"
mwan-rule 14 dst 0.0.0.0/0 group 0 remark "default rule"
!
firewall-access 10 permit outbound eth0
firewall-access 11 permit outbound ppp0
!
firewall-snat 10 overload outbound eth0
firewall-snat 11 overload outbound ppp0
!
mwan start
```

Key points:

- `mwan-group` is configured under `interface ppp0`, not under `eth1` (the physical PPPoE bearer).
- Explicit default routes are required for both the static interface and `ppp0`.
