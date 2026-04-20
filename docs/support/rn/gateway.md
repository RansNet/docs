# Release Notes (For Gateway Series)

This release notes page covers RansNet gateway series products (CMG, HSG, mfusion, mlog).

---

## 20260420-1700

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

See [DNS Filtering](../../security/webfilter.md#method-2-dns-filtering) for full configuration details.

---

**Netflow Traffic Analytics**

New Netflow dashboard in mFusion provides per-entity traffic visibility. Features include: entity-based filter, server-driven date range selection, flow records viewer with time filter, Resolve IP lookup, WiFi client columns, and an hourly raw flow cache backed by RAM for performance. Access is restricted to super-admin accounts.

---

**mFusion Account Security**

Comprehensive account security controls for the mFusion management portal:

- Configurable password policy — minimum length, complexity requirements, password history
- Account lockout after configurable failed login attempts, with automatic re-enable timer
- Forced password change on first login or by admin action
- Pre-login banners and per-notice dismissal for compliance messaging
- Full audit log for all account-related events (standardised JSON format)
- LDAP authentication phase 2 enhancements

---

**IPsec VRF Support**

IPsec tunnels can now be bound to a specific VRF, enabling traffic isolation for multi-tenant or segmented deployments. Configurable from both CLI and mFusion GUI.

---

**Hotspot Client Sticky — API Mode**

Hotspot client session stickiness now supports an API-based mode in addition to the default MySQL mode. API mode allows integration with external session management systems. The mode is selectable from the CLI.

---

### Enhancements

| Area | Change |
|---|---|
| **QoS** | Traffic shaping class number range moved to 200–279, separated from PBR range 100–199 to prevent fwmark conflicts. HTB priority formula updated accordingly. |
| **IP Track** | Enhanced system reachability tracking — improved probe interface binding and route persistence for multi-WAN environments. |
| **Show Interface** | `show interface` (no arguments) reformatted as a commercial-grade summary table; `show interface vrf` and `show ip interface` output also improved. |
| **SLA Monitoring** | SLA and system monitoring performance optimisations. |
| **Zabbix** | Interface state monitoring improvements for more accurate link-state reporting. |
| **BGP** | AS-path prepend command fix — CLI compiler updated to handle multi-word values correctly (ticket 71419). |
| **SSH** | SSH daemon hardened — weak KEX algorithms removed, MaxAuthTries tightened, ClientAlive parameters corrected. |
| **Audit Log** | Admin audit log standardised to JSON format across Device, RMM, and Hotspot modules for consistent log parsing. |
| **GUI** | Topology map tab renamed from "Topology Map" to "Topology". |
| **WiFi Pages** | Device WiFi pages use dynamic column parsing for improved compatibility across firmware versions. |
| **FTP Logs** | FTP process logs improved in Logviewer for better visibility of archive operations. |
| **Hotspot** | Auto refresh added for self-registration portal. `@` character now permitted in string filter fields. |
| **Payment** | Payment receipt is now standard — CMS toggle removed; receipt is always sent on successful transaction. |

---

### Bug Fixes

| Area | Fix |
|---|---|
| **PBR** | Fix route persistence across reloads, metric grep pattern, and tracking probe interface binding. |
| **VLAN** | Fix incorrect VLAN interface number assignment in edge cases. |
| **WireGuard** | Fix WireGuard VPN CLI command error; resolve tunnel state handling bug. |
| **FreeRADIUS** | Fix quota rejection failure when `CS-Total-MB-Weekly` / `CS-Total-MB-All` attributes carry negative values (unsigned overflow). |
| **Netflow** | Fix redirect on entity switch when no data exists; fix flow records time filter; fix loading indicator position; fix permissions for non-super-admin access. |
| **mFusion** | Fix re-enable action incorrectly resetting inactivity clock. Fix payment vendor field failing to load in CMS form. |
