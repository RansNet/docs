# Hotspot Troubleshooting (On-Premise)

The on-premise hotspot gateway depends on several components working in sequence. A failure at any layer blocks clients from completing authentication. This guide works from the network layer up — most field issues are found and resolved at Steps 1–4.

---

## Quick Checklist

Run through these five checks first before deeper investigation. Each maps to a section below.

| # | Check | Symptom if failing |
|---|---|---|
| 1 | WAN has IP address and default route | No internet from router itself |
| 2 | DNS resolves external names | Portal URL unreachable; auth fails silently |
| 3 | `firewall-snat` (MASQUERADE) is configured | Client authenticated but traffic not passing |
| 4 | RADIUS server is responding | All logins rejected; portal loops |
| 5 | Hotspot instance is started and bound to the right interface | Clients not redirected to portal |

---

## Step 1 — WAN Connectivity

Verify the WAN interface has an IP address and a default route is installed:

```
show ip interface brief
show ip route
```

Expected: the WAN interface shows a valid public or upstream IP, and `show ip route` shows a `0.0.0.0/0` default route pointing to the upstream gateway.

Then confirm the router can reach the internet:

```
ping 8.8.8.8
traceroute 8.8.8.8
```

!!! tip
    If ping to an IP succeeds but ping to a hostname fails, the issue is DNS — proceed to Step 2. If ping to an IP fails, the WAN link itself is the problem — check the upstream connection and default route.

---

## Step 2 — DNS

Configure upstream name servers and verify resolution:

```
ip name-server 8.8.8.8 8.8.4.4
```

Test that the router itself can resolve names:

```
ping google.com
ping splash.ransnet.com
```

!!! warning
    The portal hostname (e.g. `splash.ransnet.com`) must resolve correctly from the router. If it does not, the hotspot service cannot redirect clients to the login page. Verify that the configured name servers are reachable and returning valid answers.

---

## Step 3 — Firewall SNAT

On all-in-one gateway deployments (router + hotspot on the same device), a SNAT (MASQUERADE) rule is required so client traffic uses the WAN interface IP when leaving the router.

Verify the rule exists:

```
show firewall snat-list
```

If missing, add it:

```
firewall-snat 10 masquerade outbound eth0
```

Replace `eth0` with the WAN interface name. Without this rule, authenticated clients will get DHCP and pass authentication but all traffic will be dropped upstream — a common field mistake.

---

## Step 4 — RADIUS Server

Verify the RADIUS service is running and accepting requests:

```
show security radius-server
```

Test authentication directly against the server using a known working account:

```
test authentication radius-server localhost radius-key testing123 username demouser password demouser
```

Expected output: `Access-Accept` returned. If you see `Access-Reject` or a timeout, check:

- The `radius-key` (shared secret) matches the hotspot instance configuration
- The test account exists and is not expired or over quota
- The RADIUS daemon is running (`show security radius-log`)

View recent RADIUS authentication events:

```
show security radius-log
```

---

## Step 5 — Hotspot Instance

Verify the hotspot instance is configured and running, with the correct LAN and WAN interface bindings:

```
show security hotspot
show running-config begin "security hotspot"
```

Check that virtual tunnel interfaces have been created — the hotspot service moves the original LAN interface IP onto a `tun` interface (e.g. `tun0`, `tun1`) when it starts. If tunnel interfaces are absent, the service has not started successfully.

A correct minimal hotspot configuration looks like:

```
security hotspot br-vlan80
  client-dhcp-dns 8.8.8.8 8.8.4.4
  client-static 192.168.80.1 255.255.255.0
  radius-server localhost testing123
  hotspot-portal https://splash.ransnet.com/pid/ransnet/login.php
  start
```

!!! note
    The interface name in `security hotspot <iface>` must match the actual LAN interface carrying client traffic. A mismatch is one of the most common configuration mistakes — double-check this against the interface carrying the correct VLAN or bridge.

---

## Step 6 — Client-Level Verification

After confirming the service is running, validate what clients actually experience.

**Check DHCP is reaching clients**

Capture DHCP traffic on the hotspot LAN interface:

```
tcpdump interface br-vlan80 port 67
```

You should see DHCP Discover requests from clients and corresponding Offer/Ack replies from the router. Interpret the results:

| Observation | Likely cause |
|---|---|
| No DHCP Discover seen | Client not on the correct VLAN / switch port misconfigured |
| Discover seen, no Offer reply | Hotspot DHCP not active; check `client-static` and `start` |
| Offer seen, client not using it | Client has static IP or previous lease cached |

**Check DNS queries from clients**

```
tcpdump interface br-vlan80 port 53
```

DNS queries from clients should appear and receive replies. If queries appear but have no replies, the DNS forwarder is not running within the hotspot context.

**Check authenticated client sessions**

```
show security hotspot clients
```

Verify authenticated clients appear with their MAC, IP, username, session start time, and bytes transferred.

---

## Step 7 — Portal URL

The portal URL configured in `hotspot-portal` must be:

1. Resolvable by the router (tested in Step 2)
2. Reachable over HTTPS from the router
3. Exactly matching what is registered in the portal service

Test reachability from the router:

```
traceroute splash.ransnet.com
```

!!! warning
    Even a trailing `/` difference in the portal URL can cause redirect failures. Copy the URL exactly from the mFusion portal configuration. Common mistakes: `http://` instead of `https://`, missing path components, or using an unregistered PID.

---

## Symptom Reference

| Symptom | Where to look |
|---|---|
| Client gets IP but browser never redirects | Check hotspot instance is `start`ed; check tunnel interface exists |
| Browser redirects but portal page doesn't load | DNS (Step 2); portal URL mismatch (Step 7); upstream HTTPS reachable? |
| Portal loads but login always fails | RADIUS test (Step 4); wrong radius-key; account expired |
| Login succeeds but no internet after | firewall-snat missing (Step 3); check `show ip route` from hotspot context |
| Some sites work, others don't | DNS not resolving inside hotspot context; check `client-dhcp-dns` setting |
| Client disconnected immediately after login | Quota exhausted; check RADIUS account limits |
| Hotspot worked, then stopped after reboot | `start` missing from hotspot config; service not auto-starting |

---

## Common Configuration Mistakes

| Mistake | Effect | Fix |
|---|---|---|
| `firewall-snat` not configured | Clients authenticate but have no internet | Add MASQUERADE rule on WAN interface |
| Wrong interface in `security hotspot <iface>` | Clients not intercepted; no portal redirect | Match the actual LAN/VLAN interface |
| `radius-server` not configured | All logins rejected | Add `radius-server localhost <key>` |
| Portal URL uses HTTP instead of HTTPS | Portal loads but login may fail on some browsers | Use `https://` |
| `client-static` IP not in the same subnet as VLAN | DHCP assignments fail | Ensure gateway IP matches the subnet |
| `start` command missing | Service configured but not running | Add `start` at the end of the hotspot block |
