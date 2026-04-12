# Web Filtering

Web filtering controls which websites and internet services users on the network are permitted to access. Three complementary methods are available, each operating at a different layer:

| Method | How it works | Granularity | Bypass resistance |
|---|---|---|---|
| **[Firewall Rules + Objects](#method-1-firewall-rules-with-objects)** | Blocks actual TCP/UDP connections by IP, subnet, FQDN, or application signature | Rule-level — precise control per source/destination | High — blocks at connection level regardless of how DNS resolves |
| **[DNS Filtering](#method-2-dns-filtering)** | Intercepts DNS queries and returns no valid address for blocked domains | Domain-level — blocking a domain covers all its subdomains | Moderate — can be bypassed by DNS-over-HTTPS or hardcoded DNS |
| **[Cloud DNS Category Filtering](#method-3-cloud-dns-category-filtering)** | Routes DNS through a cloud resolver that enforces category-based policies | Category-level — managed lists, no manual domain maintenance | Moderate — same DNS-layer caveats apply |

These methods can be used independently or combined. A typical layered deployment uses firewall rules for application-level blocking, DNS filtering for domain-level controls, and cloud DNS for broad category enforcement.

---

## Method 1: Firewall Rules with Objects

This method uses `firewall-access` rules combined with **Firewall Objects** to permit or deny traffic based on IP addresses, subnets, FQDNs, or application signatures. Unlike DNS filtering, it blocks actual network connections — a client that resolves a domain successfully will still be denied the connection if a firewall rule matches.

**When to use this method:**

- Blocking specific applications (e.g. deny all traffic to Facebook's IP ranges)
- Denying access to known malicious IP ranges
- Permitting only approved destinations from a guest or IoT network
- Scenarios where DNS bypass (DoH, hardcoded DNS) is a concern

**How it works:**

1. Define a [Firewall Object](firewall/objects.md) containing the destinations to block or permit — using `net` (IP/subnet), `fqdn` (domain), or `app` (application, with IP lists maintained by the cloud).
2. Create a `firewall-access` rule referencing that object as the source or destination.

### GUI Configuration

Navigate to **ORCHESTRATOR → Templates → Object Groups**, create a Firewall Object with the destinations to control. Then navigate to **Device Settings → Security → Firewall Policies** and add an access rule using **Object** as the destination type.

See [Firewall Objects](firewall/objects.md) and [Firewall Policies](firewall/policies.md) for full configuration details.

### CLI Configuration

**Block a specific application by name:**

```
object-group blocked_apps
  app Facebook
  app TikTok
  app YouTube

firewall-access 100 deny outbound eth0 dst_object blocked_apps
```

**Block a set of IP ranges and domains:**

```
object-group blocked_sites
  net 203.0.113.0/24
  fqdn malicious-site.example.com

firewall-access 110 deny all dst_object blocked_sites
```

**Permit only approved destinations (default-deny approach):**

```
object-group approved_dst
  fqdn corporate.example.com
  net 10.0.0.0/8

firewall-access 100 permit outbound eth0 dst_object approved_dst
firewall-access 200 deny outbound eth0
```

**Key points:**

- `app` entries use cloud-maintained IP lists that update automatically — no manual IP management needed for well-known applications
- `fqdn` entries in objects are resolved periodically and the resulting IPs are matched against connection destinations; this differs from DNS filtering which intercepts the query itself
- Rules are evaluated top-to-bottom; place more specific permits before a broad deny

---

## Method 2: DNS Filtering

DNS filtering intercepts client DNS queries at the router and returns a controlled response — either no address (blocking the domain) or a normal upstream resolution (permitting it). Because filtering happens at the DNS layer, it requires no changes on client devices and applies to all clients on the network.

Before configuring DNS filtering, ensure the router is intercepting client DNS queries. See [DNS & DNS Rewrite](../config/dnsrewrite.md) for setup details, including the DNAT rule that transparently redirects port 53 traffic from clients regardless of what DNS server they have configured.

### Blacklisting — block specific domains

Default-permit: all domains resolve normally except those explicitly blocked.

!!! note
    Blocking a domain also blocks all its subdomains. Blocking `yahoo.com` also blocks `mail.yahoo.com`, `finance.yahoo.com`, and so on.

**CLI Configuration**

```
ip name-server 8.8.8.8 8.8.4.4
firewall-dnat 10 redirect inbound eth1 udp dport 53

ip host facebook.com reject
ip host youtube.com reject
ip host tiktok.com reject
```

To remove a block:

```
no ip host facebook.com reject
```

### Whitelisting — allow only approved domains

Default-deny: all domains are blocked unless explicitly permitted. The wildcard entry `ip host . reject` blocks everything by default; individual `ip host <domain> resolve` entries override it for approved domains.

!!! note
    Many sites and services depend on multiple domains — CDNs, authentication endpoints, API services, and analytics. Permitting a site requires permitting all its supporting domains. Use `tcpdump` or [DNS query logging](#dns-query-logging) to identify them.

**CLI Configuration**

```
ip name-server 8.8.8.8 8.8.4.4
firewall-dnat 10 redirect inbound eth1 udp dport 53

ip host . reject

ip host google.com resolve
ip host googleapis.com resolve
ip host gstatic.com resolve
ip host youtube.com resolve
ip host ytimg.com resolve
ip host ransnet.com resolve
```

To remove the default-deny and restore normal resolution:

```
no ip host . reject
```

### GUI Configuration

Navigate to **Device Settings → System**, scroll to **Static FQDN-IP Mapping**, click **+ Add**.

- To block a domain: enter the domain and set the action to **Block** (or IP `0.0.0.0`)
- To whitelist a domain under a default-deny policy: enter the domain and set the action to **Resolve**

### Identifying Required Domains

When whitelisting, use `tcpdump` on the router to capture all DNS queries clients are making while browsing a target site:

```
tcpdump interface eth1 port 53 detail
```

Watch the output as you navigate the site fully — including login, media loading, and background API calls. Add any domain that needs to resolve to the whitelist.

### Limitations

| Limitation | Detail |
|---|---|
| **DNS over HTTPS (DoH)** | Browsers can send DNS queries over HTTPS directly to a DoH provider, bypassing the router's DNS proxy entirely. |
| **DNS over TLS (DoT)** | DNS queries over TCP port 853 also bypass the DNAT redirect. Block outbound port 853 if needed. |
| **Hardcoded DNS** | Applications using a hardcoded DNS server IP bypass the DNAT redirect. |
| **VPN / Proxy** | Traffic inside a VPN or proxy tunnel bypasses DNS filtering. |

To block clients from using external DNS resolvers directly:

```
firewall-access 150 deny outbound eth0 udp dport 53
firewall-access 151 deny outbound eth0 tcp dport 53
firewall-access 152 deny outbound eth0 tcp dst 1.1.1.1,1.0.0.1,8.8.8.8,8.8.4.4 dport 443
```

Rule 150–151 block direct DNS; rule 152 blocks HTTPS to known DoH resolvers. The DNAT redirect (rule 10) continues to intercept and handle all DNS queries locally.

---

## Method 3: Cloud DNS Category Filtering

!!! note "Coming Soon"
    This section will cover category-based DNS filtering using a cloud DNS service — blocking entire categories of sites (malware, adult content, gambling, social media) without maintaining a manual domain list.
