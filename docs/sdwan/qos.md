# Traffic Shaping (QoS)

Traffic shaping (Quality of Service, QoS) controls how bandwidth is allocated across different types of traffic, ensuring critical applications receive the throughput they need while preventing any single flow or user from monopolising the link.

RansNet appliances support three complementary approaches:

| Method | What it controls | Applies to |
|--------|-----------------|------------|
| **Class-Based Shaping** | Bandwidth per traffic category | Shared pool — all matching flows share the class limit |
| **Connection-Based Limiting** | Bandwidth per individual IP address | Per-device ceiling, applied independently to each IP |
| **User-Based Control** | Bandwidth per authenticated user | Covered separately in Hotspot |

Both class-based shaping and connection-based limiting use **firewall marks** for traffic classification. This gives the same matching flexibility as firewall rules — any combination of IP address, subnet, protocol, port, FQDN, or named application object group.

![Traffic Shaping Overview](./images/traffic-qos-1.png)

In the diagram above, a 1 Gbps backhaul is shared across multiple VLANs — each typically mapped to a separate wireless SSID or LAN segment. Each VLAN is allocated a minimum guaranteed bandwidth and a burst ceiling. When one VLAN is not using its allocation, others may borrow the unused capacity up to their ceiling.

Within a VLAN, traffic can be further subdivided by application type (VLAN 30) or by individual user (VLAN 40 via Hotspot).

---

## Class-Based Shaping

Class-based shaping uses **HTB (Hierarchical Token Bucket)** queuing to allocate bandwidth across defined traffic classes. Each class has a guaranteed minimum `rate`, a maximum `ceil`, and a burst scheduling `prio`. Under congestion, every class receives its guaranteed `rate` unconditionally; when spare capacity exists, classes borrow toward their `ceil` in priority order.

Typical use cases:

- Guarantee bandwidth for business-critical applications (video conferencing, VoIP, ERP/CRM)
- Cap bandwidth for non-business traffic (social media, streaming) without blocking it entirely
- Prioritise real-time traffic over bulk transfers during peak hours
- Differentiate service levels between VLANs or departments sharing the same uplink

### How Traffic is Classified

Classification is a two-step process:

1. A `firewall-set` rule inspects traffic and stamps matching packets with a numeric **fwmark**.
2. A QoS `class` references the same mark number — the scheduler places marked packets into the corresponding class queue.

Because firewall rules handle all the matching, QoS classes inherit the full firewall matching vocabulary, including named application object groups and FQDN.

!!! note
    When configuring via the GUI or orchestrator, `firewall-set` rules and fwmark assignments are created automatically — no manual mark management is required. The orchestrator derives the fwmark from the class number (e.g. class `100` → upload mark `1001`, download mark `1002`). The CLI reflects these auto-generated values in `show running-config`.

!!! warning "Avoid Overlapping firewall-set Rules Across Classes"
    `firewall-set` rules are evaluated in ascending rule number order. Each matching rule **overwrites** the fwmark set by any previous rule — the last matching rule wins. If two classes have `firewall-set` rules that can match the same packet, the packet will always end up in the class corresponding to the higher-numbered rule, regardless of which class was intended.

    For example, if rule `1001` matches `tcp dport 443` (HTTPS) and rule `1501` matches a social media object group that includes HTTPS destinations, any HTTPS traffic to a social media site will be marked `1501` and placed into class `150`, not class `100`.

    To avoid this:

    - Use **mutually exclusive** match criteria across classes wherever possible.
    - Where overlap is unavoidable, place the **more specific rule at a higher number** so it takes precedence (overwrites the broader mark). This is the same discipline required for PBR rules.

### Upload and Download

QoS acts on **egress** (outgoing) traffic only. To shape both directions, configure `traffic-shape` on both the WAN and LAN interfaces:

- **Upload** (client → internet): configure on the **WAN interface** (`eth0`, `wwan0`, etc.)
- **Download** (internet → client): configure on the **LAN interface** (`vlan 1`, `vlan 2`, etc.)

Separate fwmarks are used for each direction — the upload mark matches on **destination** port or object (traffic going to the internet), and the download mark matches on **source** port or object (traffic returning from the internet). The orchestrator assigns these marks automatically following the class number convention.

### Bandwidth Parameters and Priority

Each traffic class is defined by three parameters. The GUI labels them **Minimum**, **Maximum**, and derives priority from the **Class No.** automatically. The CLI takes them as positional arguments. Their underlying HTB equivalents are shown in parentheses for reference.

| GUI label | CLI position | HTB term | Description |
|---|---|---|---|
| **Minimum** (Kbps) | 1st argument after class no. | `rate` | Guaranteed bandwidth floor — always delivered |
| **Maximum** (Kbps) | 2nd argument after class no. | `ceil` | Burst ceiling — maximum including borrowed capacity |
| *(derived from Class No.)* | — | `prio` | Burst scheduling order — which class borrows first |

#### Minimum — Guaranteed Bandwidth

**Minimum** is the bandwidth a class is always entitled to receive, unconditionally. The HTB scheduler reserves this allocation regardless of what other classes are doing. Even when the link is fully saturated and every class is competing, each class will receive at least its configured Minimum.

Use Minimum to protect latency-sensitive traffic — VoIP, real-time video, business-critical SaaS — from being crowded out during peak hours.

#### Maximum — Burst Ceiling

**Maximum** is the upper limit of bandwidth a class can ever consume, including capacity borrowed from the parent pool. A class can only exceed its Minimum up to the Maximum when other classes are using less than their own Minimum and spare capacity is available.

- **Minimum = Maximum** — creates a hard cap. The class never borrows and is strictly limited to its guaranteed allocation.
- **Maximum > Minimum** — allows opportunistic bursting. The class expands to fill idle capacity up to the ceiling.

#### Priority — Burst Scheduling Order

Priority (`prio`) controls which class gets first access to spare capacity once all class Minimums have been satisfied. It is **not directly configurable** — it is derived automatically from the Class No. using `(Class No. − 100) ÷ 10`. A lower class number gives a higher burst priority (borrows spare bandwidth before higher-numbered classes).

!!! note "Priority is burst-order only — not strict priority"
    Priority only governs the order in which classes borrow spare capacity **above** their Minimum. It has no effect on guaranteed allocations. A class with the lowest priority (class no. 170–179) and a Minimum of 50 Mbps will always receive its full 50 Mbps even when a highest-priority class is simultaneously active. True strict priority — "always drain this class before touching any other" — is not a feature of HTB.

| Class No. range | Priority tier | Burst order |
|-------------|----------|----------------|
| 100–109 | 0 | Highest — borrows spare capacity first |
| 110–119 | 1 | |
| 120–129 | 2 | |
| 130–139 | 3 | |
| 140–149 | 4 | |
| 150–159 | 5 | |
| 160–169 | 6 | |
| 170–179 | 7 | Lowest — borrows last |

#### Order of Operations

HTB processes each scheduling cycle in two sequential phases:

1. **Guarantee phase** — all active classes are serviced up to their Minimum. This occurs unconditionally and in parallel — priority has no influence here. Every class receives its reserved allocation regardless of class number.

2. **Borrow phase** — if spare capacity remains after all Minimums are satisfied, HTB offers the excess to classes in priority order. The highest-priority class (lowest class number) borrows first up to its Maximum, then the next tier, and so on. If the link is fully consumed in the guarantee phase, the borrow phase does not occur and priority has no effect at all.

| Condition | Which phase applies | Is priority effective? |
|---|---|---|
| Link not fully loaded | Borrow phase — classes burst toward Maximum | Yes — lower class number bursts first |
| Link saturated, all classes at Minimum | Guarantee phase only | No — all classes receive their Minimum |
| A class has Minimum = 0 | Must always borrow; nothing guaranteed | Partially — risks starvation under congestion |

#### Choosing Minimum and Maximum

- Set **Minimum** to the bandwidth the application must have to function acceptably under worst-case congestion — this is its hard protection floor.
- Set **Maximum** to the most bandwidth you are willing to allow the class to consume when the link is quiet.
- The gap between Minimum and Maximum is the **burst window** — how aggressively the class expands into spare capacity when available.
- To create a strict cap with no bursting, set **Minimum = Maximum**.

#### Class Number and Filter Order

The **Class No.** also controls **filter evaluation order**, independent of HTB scheduling. Lower class numbers are matched first. If a packet could satisfy multiple class filters, it is placed into the lowest-numbered matching class. Assign lower class numbers to traffic types that should take precedence in classification.

### GUI Configuration

Navigate to **Device Settings → SD-WAN → Traffic Shaping** and click the **Class-Based Shaping** tab.

![Class-Based Shaping](./images/traffic-qos-2.png)

### CLI Configuration

```
!
hostname HSA-520
!
interface eth0
 description "Connection to WAN"
 enable
 ip address dhcp
 traffic-shape 10000000 10000000
  class 100 5000 10000 fwmark 1001 remark class-100-upload
  class 120 3000 4000 fwmark 1201 remark class-120-upload
  default bandwidth 512 512
!
interface eth1
 description "Do NOT configure"
 enable
!
interface wwan0
 enable
 traffic-shape 10000000 10000000
  class 110 7000 8000 fwmark 1101 remark class-110-upload
!
interface vlan 1 1
 description "Default VLAN for all LAN ports"
 enable
 ip address 192.168.8.1/22
 dhcp-server
  router 192.168.8.1
  dns 8.8.8.8 8.8.4.4
  range 192.168.8.10 192.168.11.254
  enable
 traffic-shape 10000000 10000000
  class 100 5000 10000 fwmark 1002 remark class-100-download
  class 110 5000 6000 fwmark 1102 remark class-110-download
  class 120 1000 2000 fwmark 1202 remark class-120-download
  default bandwidth 512 512
!
ip name-server 8.8.8.8 8.8.4.4
!
object-group VoIP
 net 118.189.175.168
!
object-group firewall_obj
 net 2.2.2.2
 app WhatsApp
 fqdn docs.ransnet.com
!
object-group social_sites
 app Facebook
 app Tiktok
 app YouTube
!
firewall-input 100 permit all tcp src 192.168.8.0/22 dport 22
!
firewall-limit 3001 2000 all ip dst_object firewall_obj remark 300-upload
firewall-limit 3002 2000 all ip src_object firewall_obj remark 300-download
!
firewall-access 100 permit outbound eth0
firewall-access 101 permit outbound wwan+
!
firewall-snat 100 overload outbound eth0
firewall-snat 101 overload outbound wwan+
!
firewall-set 1001 mark 1001 access ip dst_object VoIP remark class-100-upload
firewall-set 1101 mark 1101 access tcp dport 443 remark class-110-upload
firewall-set 1102 mark 1102 access tcp sport 443 remark class-110-download
firewall-set 1201 mark 1201 access ip dst_object social_sites remark class-120-upload
firewall-set 1202 mark 1202 access ip src_object social_sites remark class-120-download
```

**Key points:**

- `traffic-shape <min> <max>` sets the root class bandwidth envelope in Kbps. Use `10000000` (10 Gbps) when there is no overall interface cap — individual class limits do the actual shaping.
- `class <no> <min> <max> fwmark <mark>` defines a traffic class. `no` (Class No.) determines both filter evaluation order and burst priority — lower is higher priority. `min` is the guaranteed Minimum; `max` is the Maximum burst ceiling.
- `fwmark <mark>` links the class to a `firewall-set` rule carrying the same mark number. When using the orchestrator or GUI, this value is assigned automatically — no manual configuration needed.
- `default bandwidth <rate> <ceil>` creates a catch-all class for traffic that does not match any fwmark. Omitting it allows unmatched traffic to pass at wire speed, which is acceptable when only per-class ceilings are needed.
- `remark` is a free-text label stored in the running config for identification only — it has no effect on shaping behaviour.

!!! tip
    For traffic that must be protected under congestion (VoIP, real-time video), set a meaningful **Minimum** — this is the only mechanism that guarantees bandwidth when the link is saturated. Assign lower Class Numbers to traffic that should burst first when spare capacity is available, and use **Maximum** to cap the most each class can consume.

!!! note
    `firewall-set` rules are evaluated for all traffic passing through the device. Only packets that match a rule receive a mark; all other traffic is unclassified. If a `default bandwidth` class is not configured, unclassified traffic exits at wire speed outside of HTB's control.

### Verification

Example output:

```
HSA-520# show interface traffic-shape 

Interface: eth0
  Class    Min         Max         Prio    FW Mark           Sent              Dropped
  -------  ----------  ----------  ------  ----------------  ----------------  -------
  100      5Mbit       10Mbit      0       1001/0x3e9        0B/0p             0
  120      3Mbit       4Mbit       2       1201/0x4b1        0B/0p             0
  default  512Kbit     512Kbit     -       (catch-all)       295439B/2692p     0

Interface: vlan1
  Class    Min         Max         Prio    FW Mark           Sent              Dropped
  -------  ----------  ----------  ------  ----------------  ----------------  -------
  110      5Mbit       6Mbit       1       1102/0x44e        0B/0p             0
  100      5Mbit       10Mbit      0       1002/0x3ea        0B/0p             0
  120      1Mbit       2Mbit       2       1202/0x4b2        0B/0p             0
  default  512Kbit     512Kbit     -       (catch-all)       0B/0p             0

Interface: wwan0
  Class    Min         Max         Prio    FW Mark           Sent              Dropped
  -------  ----------  ----------  ------  ----------------  ----------------  -------
  110      7Mbit       8Mbit       1       1101/0x44d        0B/0p             0
  [i] No default class — unclassified traffic passes at wire speed (uncapped)
HSA-520# 
```

**Min** is the guaranteed bandwidth floor (Minimum); **Max** is the burst ceiling (Maximum). **FW Mark** shows the decimal value (matching the `firewall-set` rule number) alongside the kernel's internal hex representation. **Prio** is the burst scheduling order (0–7), derived automatically from the Class No. using `(Class No. − 100) ÷ 10` — class `100` → prio `0` (highest), class `179` → prio `7` (lowest).

For low-level inspection of the underlying `qos` state:

```
HSA-520# show interface traffic-class eth0     (HTB class details and statistics)
HSA-520# show interface traffic-filter eth0    (tc filter entries and fwmark handles)
HSA-520# show interface traffic-control        (qdisc state for all interfaces)
```

---

## Connection-Based Limiting (rate-limit)

Connection-based limiting enforces a per-IP bandwidth cap. Unlike class-based shaping — which is a shared pool across all matching flows — rate-limit applies independently to each source and destination IP address.

Typical use cases:

- Prevent any single device from saturating the internet link
- Enforce per-user fair-use policies on guest or hotspot networks
- Rate-limit access to a specific application or destination on a per-client basis

### GUI Configuration

Navigate to **Device Settings → SD-WAN → Traffic Shaping** and click the **Connection-Based Limiting** tab.

![Connection-Based Limiting](./images/traffic-qos-3.png)

### CLI Configuration

```
object-group firewall_obj
 app WhatsApp
 fqdn docs.ransnet.com
 net 2.2.2.2
!
firewall-limit 3001 2000 all ip dst_object firewall_obj remark limit-upload
firewall-limit 3002 2000 all ip src_object firewall_obj remark limit-download
```

**Key points:**

- `firewall-limit <id> <rate_kbps>` creates a rate-limiting rule. `id` sets evaluation order — lower ID is evaluated first. `rate_kbps` is the per-IP ceiling in Kbps.
- Configure both an upload rule (`dst_object`) and a download rule (`src_object`) to limit traffic in both directions.
- The same object-group vocabulary applies: match by IP, subnet, FQDN, protocol, port, or application.

### Verification

Example output:

```
HSA-520# show firewall limit-list

###### Summary of Rate limit chain ########
 pkts bytes target         prot opt in   out   source     destination
    0     0 LIMITBPS-3001  all  --  *    *     0.0.0.0/0  0.0.0.0/0  match-set firewall_obj dst
    0     0 LIMITBPS-3002  all  --  *    *     0.0.0.0/0  0.0.0.0/0  match-set firewall_obj src

###### Chain-LIMITBPS-3001 details ###########
Chain LIMITBPS-3001 (1 references)
 pkts bytes target  prot opt in   out   source     destination
    0     0 DROP    all  --  *    *     0.0.0.0/0  0.0.0.0/0  limit: above 500000b/s mode dstip
    0     0 DROP    all  --  *    *     0.0.0.0/0  0.0.0.0/0  limit: above 500000b/s mode srcip
    0     0 ACCEPT  all  --  *    *     0.0.0.0/0  0.0.0.0/0

###### Chain-LIMITBPS-3002 details ###########
Chain LIMITBPS-3002 (1 references)
 pkts bytes target  prot opt in   out   source     destination
    0     0 DROP    all  --  *    *     0.0.0.0/0  0.0.0.0/0  limit: above 500000b/s mode dstip
    0     0 DROP    all  --  *    *     0.0.0.0/0  0.0.0.0/0  limit: above 500000b/s mode srcip
    0     0 ACCEPT  all  --  *    *     0.0.0.0/0  0.0.0.0/0
```

Each chain contains two DROP rules — one keyed on source IP (`srcip`) and one on destination IP (`dstip`) — so the per-IP cap is enforced for each endpoint independently. Packets exceeding the rate are dropped before forwarding; conforming traffic reaches the ACCEPT rule and is forwarded normally.

---

## Design Notes

### Class Numbering Convention

Valid class numbers are in the range **100–179**. The class number determines both the scheduling priority tier and the auto-assigned fwmark values. Use the table below to select the right class range for the desired priority:

| Class No. range | Burst priority (Prio) | Burst order | Typical use |
|-------------|----------|----------------|-------------|
| 100–109 | 0 | Highest | VoIP, real-time video |
| 110–119 | 1 | | Video conferencing |
| 120–129 | 2 | | Business-critical (SaaS, ERP) |
| 130–139 | 3 | | Interactive web (HTTPS) |
| 140–149 | 4 | | General web browsing |
| 150–159 | 5 | | File downloads |
| 160–169 | 6 | | Social media, streaming |
| 170–179 | 7 | Lowest | Background / bulk transfers |

The orchestrator automatically derives fwmark values from the class number — class `N` gets upload mark `N×10+1` and download mark `N×10+2`:

| Burst priority | Example Class No. | Auto upload mark | Auto download mark |
|----------|---------------|-----------------|-------------------|
| 0 (highest) | 100 | 1001 | 1002 |
| 1 | 110 | 1101 | 1102 |
| 2 | 120 | 1201 | 1202 |
| 3 | 130 | 1301 | 1302 |
| 4 | 140 | 1401 | 1402 |
| 5 | 150 | 1501 | 1502 |
| 6 | 160 | 1601 | 1602 |
| 7 (lowest) | 170 | 1701 | 1702 |

### Combining Methods

Class-based shaping and connection-based limiting work simultaneously and complement each other:

- **Class-based shaping** caps the *total* bandwidth for a traffic category — e.g. all social media traffic across all users combined ≤ 10 Mbps.
- **Connection-based limiting** caps each *individual device* within that category — e.g. each device ≤ 2 Mbps of social media, regardless of what others are doing.

Applying both gives aggregate control plus per-user fairness with a single configuration.

!!! note
    Per-connection means a specific 5-tuple (source IP, destination IP, protocol, source port, destination port). One device typically opens many connections simultaneously. Class-based shaping controls the aggregate of all those connections; connection-based limiting controls the total for each device IP.
