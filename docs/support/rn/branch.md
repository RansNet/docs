# Release Notes (For Branch Series)

This release notes page covers RansNet branch series products (HSA, UA, XE, UAP).

---

## 20260420-0050

### New Features

**DNS Filtering**

New `dns-group` and `dns-filter` CLI commands provide domain-based DNS filtering with blacklist and whitelist modes. Domain entries automatically cover all subdomains (wildcard matching); more specific entries take precedence over broader ones. Named groups (`dns-group`) allow bulk domain management — add or remove domains from a group without modifying filter rules.

```
dns-group adult-sites
  domain playboy.com
!
dns-filter 100 deny group adult-sites
dns-filter 200 permit all
```

See [DNS Filtering](../security/webfilter.md#method-2-dns-filtering) for full configuration details.

---

**EasyMesh Multi-Fronthaul Support**

EasyMesh controller now supports multiple fronthaul SSIDs across different VLANs. Maximum BSS count is derived dynamically from the radio configuration rather than being hardcoded. Combined with improved agent upstream tracking, this enables more flexible mesh deployments with per-VLAN fronthaul segmentation.

---

**IPsec VRF Support**

IPsec tunnels can now be bound to a specific VRF for traffic isolation. VRF assignment is supported for both standard IPsec and VTI-mode tunnels. VTI return routes now use an explicit next-hop for correct per-VRF routing.

---

**Fibocom Modem Support**

Provider registration updated to support Fibocom FM160 and FG160 modems, in addition to existing Quectel RG520N support.

---

### Enhancements

| Area | Change |
|---|---|
| **QoS** | Traffic shaping class number range moved to 200–279, separated from PBR range 100–199 to prevent fwmark conflicts. HTB priority formula updated accordingly. |
| **IP Track** | Enhanced system reachability tracking — improved probe interface binding and route persistence for multi-WAN environments. |
| **EasyMesh** | `no interface wifi mesh` now fully cleans up all EasyMesh state: clears `ieee1905managed` flags, removes `/tmp/ezmesh-*.conf` runtime files, and restores wsplcd/ezmesh configs to defaults. |
| **EasyMesh Agent** | Upstream tracking optimised — bridge state and veth cross-netns handling improved for more reliable backhaul detection after wifi restart. |
| **PBR** | PBR features optimised; `show ip pbr` output improved. |
| **Show Interface** | `show interface vrf`, `show ip interface`, and `show interface wwan` output improved for clarity and consistency. |
| **SLA Monitoring** | SLA and system monitoring performance optimisations. |
| **Interface Monitoring** | Enhanced interface state monitoring for improved link-state accuracy. |
| **SSH** | SSH daemon hardened — weak KEX algorithms removed, MaxAuthTries tightened, ClientAlive parameters corrected. |
| **IPsec VTI** | VTI return route now uses `ip route replace` with explicit via next-hop for reliable per-VRF forwarding. |

---

### Bug Fixes

| Area | Fix |
|---|---|
| **EasyMesh Agent** | Fix CAP↔NonCAP state cycle causing agent registration failure on first boot. |
| **EasyMesh Agent** | Fix stuck `mapsig-` SSIDs — WPS re-trigger and bitwise MAP capability checks corrected. |
| **EasyMesh Agent** | Fix spurious VLAN/eth0 bridge created on first boot. |
| **EasyMesh Agent** | Fix `ubus` interface calls — use UCI interface name (`lan`) and correct `down`/`up` method names. |
| **EasyMesh Controller** | Fix fronthaul VAP (`mbox`) disappearing when the managed VLAN matches the management network. |
| **BGP** | Fix XE300 unable to recognise BGP `show` commands (ticket 71874). |
| **IPsec** | Fix 7 bugs identified in code review — covering tunnel teardown, status parsing, and VTI route handling. |
| **WireGuard** | Fix VRF interface display issue in `show` output. |
| **P2P Interface** | Fix P2P interface handling in PBR routing. |
