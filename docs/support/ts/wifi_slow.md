# Wi-Fi Slow Issues

## Qualify the Problem First

Before troubleshooting, confirm you are dealing with a performance problem — not a coverage problem.

| Symptom | Action |
|---|---|
| Weak Wi-Fi signal and slow speed | **Stop here.** Focus on Wi-Fi coverage — add APs or ask users to move closer to the nearest AP. |
| Strong Wi-Fi signal but slow speed | Continue with this guide. |

This guide assumes users have adequate signal strength and addresses the case where measured throughput is significantly lower than the configured or provisioned bandwidth.

---

## Understanding the Problem Space

An end-to-end Wi-Fi experience passes through multiple components. A bottleneck at any single layer degrades the experience for all users. Jumping straight to the most visible component (the HotSpot Gateway) without isolating the layer wastes time and risks missing the real cause.

The four potential bottlenecks, from outer to inner:

```
Internet ──► ISP Link ──► Internet Router ──► HotSpot Gateway ──► Wi-Fi Infrastructure ──► Client
              Step 1          Step 2               Step 3                  Step 4
```

Work through the steps in order. Stop as soon as you find the failing layer — there is no value diagnosing deeper layers until the outer one is resolved.

---

## Step 1 — ISP Link

**Goal:** Confirm the ISP is delivering the contracted bandwidth.

Disconnect the ISP link from the router and connect it directly to a test PC (configure the PC with the correct IP, gateway, and DNS for that link). Run a speed test.

| Result | Action |
|---|---|
| Speed ≈ contracted SLA | ISP link is healthy. Proceed to Step 2. |
| Speed significantly below SLA | Escalate to the ISP. Do not proceed until the link is confirmed healthy. |

!!! note
    Non-guaranteed links (e.g. broadband, shared fibre) may deliver contracted speeds during this test but still underperform during peak hours. If the ISP passes here but users consistently complain at specific times of day, note those times and escalate to the ISP with timestamp evidence.

---

## Step 2 — Internet Router

**Goal:** Rule out the Internet router as a bottleneck. Skip this step if the HSG is also acting as the Internet router.

**2a — Check WAN utilisation**

Read the current WAN port bandwidth utilisation on the router (call this **Y Mbps**).

- If **Y ≈ X** (contracted speed): the link is saturated. No tuning on the router will help — escalate to the customer to upgrade the ISP link.
- If **Y << X**: the link has headroom. Continue to 2b.

**2b — Test router throughput under live load**

Connect a test PC to a spare LAN port on the router. Run a speed test **without** removing any other connections — the live user traffic must remain active during the test.

Expected: `PC speed test result + Y ≈ X`

If the sum is significantly less than X, the router CPU or NAT table is the bottleneck. Upgrade the router and retest.

!!! note
    Many routers achieve near-line-rate throughput on a clean bench test but degrade significantly under real load because connection tracking, NAT, and stateful inspection are CPU-intensive. Always test with live user traffic present.

If user experience improves after the upgrade, stop here. Otherwise, proceed to Step 3.

---

## Step 3 — HotSpot Gateway (HSG)

**Goal:** Isolate HSG performance as a cause. This step covers performance only — for service availability issues, see [Hotspot Troubleshooting (On-Premise)](hotspot_onprem.md).

**3a — Check WAN utilisation**

Read the HSG WAN port utilisation (call this **Y Mbps**).

- If **Y ≈ X**: link is saturated. Escalate to the customer to upgrade the ISP link.
- If **Y << X**: continue to 3b.

**3b — Test HSG routing throughput**

Connect a test PC to a spare LAN port on the HSG configured as a plain routed port (hotspot **disabled** on this port). Run a speed test with live user traffic remaining active.

Expected: `PC speed test result + Y ≈ X`

If the result is poor, the HSG is the bottleneck. Upgrade the HSG model and retest. Stop here once the test passes.

**3c — Test HSG hotspot throughput**

Connect a test PC to a switch port on the hotspot VLAN and run a speed test with live user traffic active.

If throughput is poor (similar to what end users report, or well below the configured per-user bandwidth), try the following remediation options — apply one at a time and retest after each:

| Option | Action |
|---|---|
| Upgrade hardware | Replace with a higher-capacity HSG model |
| Reduce VLAN size | Split the hotspot subnet into multiple smaller instances across separate VLANs |
| Disable optional services | Stop MAC accounting (`stop macc`), set `client-local-dns off`, stop the syslog server if running locally |

If user experience improves, stop here. Otherwise, proceed to Step 4.

---

## Step 4 — Wi-Fi Infrastructure

**Goal:** Identify AP, PoE switch, or radio configuration issues.

Wi-Fi infrastructure problems typically present in one of two ways:

| Symptom | Likely cause |
|---|---|
| Clients unable to obtain IP address | AP power issue; excessive inter-AP interference causing clients to bounce |
| Connected clients have high latency or low throughput | AP overload; PoE switch under-spec; radio misconfiguration |

**Verify the AP layer in isolation:** ping the default gateway from a client over Wi-Fi only (before traffic reaches the HSG). If latency is already high at this stage, the problem is in the Wi-Fi layer — not the HSG or upstream network.

### Power Supply

- Ensure the UPS has sufficient capacity for all connected equipment. An undersized or ageing UPS can cause subtle performance degradation or intermittent hangs even when all devices appear powered on.
- Ensure the PoE switch meets the actual power draw of all connected APs. Some budget switches advertise a total PoE budget that cannot be sustained when all ports are active simultaneously — APs may remain online but at degraded performance. As a rule of thumb, if you must use a budget PoE switch, do not populate more than 70% of its PoE ports (e.g. use 16 of 24 ports). Use additional switches rather than overloading a single one.

### PoE Switch Performance

The PoE switch carries all AP traffic in addition to powering the APs. Verify it is not a throughput bottleneck — particularly in deployments using VLANs or inter-AP roaming, where the switch processes significant traffic.

### Radio Configuration

Use a controller-based AP system (on-premise or cloud) to apply the following settings. These are the most impactful levers for dense deployments:

| Setting | Recommendation |
|---|---|
| **Channel assignment** | Assign non-overlapping channels to adjacent APs. On 2.4 GHz use channels 1, 6, 11. On 5 GHz use automatic channel planning from the controller where available. |
| **Transmit power** | Reduce AP transmit power in dense environments. High power causes co-channel interference — nearby APs hear each other loudly, contend for airtime, and throughput drops. Aim for the minimum power that provides adequate coverage for the intended cell. |
| **Client roaming (802.11r/k/v)** | Enable fast BSS transition (802.11r), neighbour reports (802.11k), and BSS transition management (802.11v). Without these, clients hold on to a distant AP rather than roaming to a closer one — the result is good signal on the AP but poor throughput due to a weak reverse path. |
| **Load balancing** | Enable band steering (push capable clients to 5 GHz) and AP load balancing in the controller. In crowded areas this prevents a single AP from becoming overloaded while adjacent APs are underutilised. |

!!! tip
    For complex radio environment tuning — especially in venues with many APs in a small area — engage your AP vendor's RF planning team. Automated RF planning tools built into enterprise controllers (Cisco, Aruba, Ruckus, Fortinet, etc.) can survey the environment and recommend channel and power settings that are difficult to derive manually.
