# NetFlow API

## Overview

NetFlow is an industry-standard network protocol (originally developed by Cisco) for collecting and exporting information about IP traffic flows passing through a network device. Each **flow record** captures the key attributes of a conversation between two endpoints: source and destination IP, source and destination port, protocol, byte and packet count, duration, and TCP flags. By aggregating these records, a NetFlow collector can reconstruct a detailed picture of who is talking to whom, how much traffic each conversation generates, and what protocols are in use — without capturing actual packet payloads.

The mfusion platform acts as a **NetFlow collector**, receiving flow records exported by RansNet routers and third-party devices using standard NetFlow v5/v9 or IPFIX export. Collected data is stored in **nfcapd** capture files (one file per hour) and indexed for fast retrieval. The NetFlow API provides programmatic access to this data, enabling system integrators and developers to build custom dashboards, security monitoring tools, and traffic analytics applications on top of the mfusion infrastructure.

The API exposes four capabilities:

| Endpoint | Method | Description |
|---|---|---|
| `/mbox/api/netflow/auth` | POST | Authenticate and obtain a JWT bearer token |
| `/mbox/api/netflow/latest` | GET | Retrieve the time range of the most recent capture file |
| `/mbox/api/netflow/kpi` | GET | Retrieve aggregated KPI metrics and traffic statistics |
| `/mbox/api/netflow/flows` | GET | Retrieve individual flow records for a time range |
| `/mbox/api/netflow/resolve` | GET | Batch reverse DNS lookup — resolve IPs to FQDNs |

---

## Authentication

All API requests must include a valid JWT bearer token obtained from the auth endpoint. Tokens are scoped per entity and per user.

### Enabling API Access

API access must be enabled before credentials can be generated:

1. Navigate to **Admin → General → mFusion API** tab in the mBox GUI.
2. Use the **Entity Search Box** to locate the target entity.
3. Under the **Netflow API** column, toggle the switch to enable API access for the entity.
4. Click the **Users** icon beside the toggle to manage user-level access.
5. Locate the desired user, enable their **Netflow API** toggle, then click the **key icon** to retrieve their **API ID** and **Secret Key**.

To rotate credentials, click **Request New Access ID and Secret**. This invalidates the previous credentials immediately.

### Obtaining a Token

```
POST /mbox/api/netflow/auth
Content-Type: application/json

{
  "api_id": "your_api_id",
  "secret": "your_secret_key"
}
```

Include the returned token in all subsequent requests:

```
Authorization: Bearer {your_token}
```

!!! note
    For full authentication details including token expiry and error codes, refer to Chapter 2.1 of the RansNet CLIENT API v3.1.0 Developer Reference Manual.

---

## Response Format

All endpoints return JSON. Every response includes a `status` field indicating success or failure.

**Successful response:**

```json
{
  "status": true,
  "result": { ... }
}
```

**Failed response:**

```json
{
  "status": false,
  "errorMessage": "Token validation failed"
}
```

| Field | Type | Description |
|---|---|---|
| **status** | Boolean | `true` if the request succeeded; `false` if it failed |
| **result** | Object or Array | Response payload (only present on success) |
| **errorMessage** | String | Reason for failure (only present on failure) |

---

## API Endpoints

### Get Latest File Range

Returns the time range of the most recently written nfcapd capture file for the specified entity. Use this endpoint first to discover the available data window before querying KPI or flow data — particularly useful in automated integrations where the current time boundary is unknown.

**Base URL:** `https://YOUR_SERVER_ADDRESS/mbox/api/netflow/latest`

#### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| **entity** | String | No | Data source entity: `netflow` or `lentor`. Default: `netflow` |

#### Example Request

```
GET /mbox/api/netflow/latest?entity=netflow
Authorization: Bearer {your_token}
```

#### Example Response

```json
{
  "status": true,
  "result": {
    "start":          "2026/03/25 09:00",
    "end":            "2026/03/25 09:59",
    "latest_file":    "nfcapd.202603250900",
    "interval_sec":   3600,
    "interval_label": "1h",
    "is_cached":      true,
    "data_start":     "2026/03/20 00:00",
    "data_end":       "2026/03/25 09:59"
  }
}
```

#### Response Fields

| Field | Type | Description |
|---|---|---|
| **start** | String | Start timestamp of the latest capture file |
| **end** | String | End timestamp of the latest capture file |
| **latest_file** | String | nfcapd filename (format: `nfcapd.YYYYMMDDHHmm`) |
| **interval_sec** | Integer | Capture file interval in seconds (typically `3600` = 1 hour) |
| **interval_label** | String | Human-readable interval label (e.g. `"1h"`) |
| **is_cached** | Boolean | Whether the result was served from the hourly cache |
| **data_start** | String | Earliest available data timestamp across all stored files |
| **data_end** | String | Latest available data timestamp |

---

### Get KPI & Traffic Statistics

Returns aggregated traffic metrics, top talkers, protocol distribution, and time-series data for a specified time window. Data is served from pre-computed hourly cache files where available. For the current in-progress hour, the API falls back to a live `nfdump` query.

**Base URL:** `https://YOUR_SERVER_ADDRESS/mbox/api/netflow/kpi`

#### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| **start** | String (`YYYY/MM/DD HH:mm`) | Yes | Start of the query time window |
| **end** | String (`YYYY/MM/DD HH:mm`) | Yes | End of the query time window |
| **entity** | String | No | Data source entity: `netflow` or `lentor`. Default: `netflow` |

#### Response Fields

| Field | Type | Description |
|---|---|---|
| **meta.start** | String | Query start time as provided |
| **meta.end** | String | Query end time as provided |
| **meta.source** | String | `"hourly_cache"` — served from cache; `"fallback"` — live nfdump query |
| **kpi.total_traffic** | String | Human-readable total volume (e.g. `"169 G"`, `"2.5 G"`) |
| **kpi.total_bytes** | Integer | Raw byte count for the period |
| **kpi.active_flows** | Integer | Total number of flow records in the period |
| **kpi.avg_throughput_mbps** | Number | Average throughput in Mbps across the query window |
| **kpi.denied_flows** | Integer | Count of flows with RST flag set (rejected connections) |
| **top_src** | Array | Top 10 source IPs by bytes — each entry: `ip`, `bytes`, `packets`, `flows` |
| **top_dst_ip** | Array | Top 10 destination IPs by bytes — each entry: `ip`, `bytes`, `packets`, `flows` |
| **top_dst** | Array | Top 10 destination ports by flows — each entry: `port`, `label`, `flows`, `bytes` |
| **protocols** | Array | Protocol share breakdown — each entry: `proto`, `proto_num`, `bytes`, `flows`, `pct` |
| **timeseries** | Array | Hourly data points — each entry: `timestamp`, `bytes`, `flows` |

#### Example: Single Hour

```
GET /mbox/api/netflow/kpi?start=2026/03/25%2009:00&end=2026/03/25%2009:59&entity=netflow
Authorization: Bearer {your_token}
```

```json
{
  "status": true,
  "result": {
    "meta": {
      "start":  "2026/03/25 09:00",
      "end":    "2026/03/25 09:59",
      "source": "hourly_cache"
    },
    "kpi": {
      "total_traffic":       "169 G",
      "total_bytes":         181500000000,
      "active_flows":        4239914,
      "avg_throughput_mbps": 382.4,
      "denied_flows":        626507
    },
    "top_src": [
      { "ip": "10.65.30.2", "bytes": 532000000000, "packets": 1082276700, "flows": 10393611 }
    ],
    "protocols": [
      { "proto": "TCP", "proto_num": 6, "bytes": 180000000000, "flows": 4200000, "pct": 99.5 }
    ],
    "timeseries": [
      { "timestamp": "2026-03-25T09:00:00Z", "bytes": 181500000000, "flows": 4239914 }
    ]
  }
}
```

#### Example: Date Range (Multiple Hours)

```
GET /mbox/api/netflow/kpi?start=2026/03/25%2000:00&end=2026/03/25%2009:59&entity=netflow
Authorization: Bearer {your_token}
```

When the query spans multiple hours, `timeseries` returns one data point per hour, and KPI values are aggregated across the full range.

---

### Get Detailed Flow Records

Returns individual NetFlow records for a specified time window. Each record represents a single network conversation — its endpoints, protocol, volume, duration, and a firewall verdict derived from TCP flags. Records are sorted newest-first.

**Base URL:** `https://YOUR_SERVER_ADDRESS/mbox/api/netflow/flows`

!!! note
    The `limit` parameter is capped at **200 records per request** regardless of the value provided. To retrieve larger datasets, page through the time window by issuing multiple requests with narrower `start`/`end` ranges.

#### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| **start** | String (`YYYY/MM/DD HH:mm`) | Yes | Start of the query time window |
| **end** | String (`YYYY/MM/DD HH:mm`) | Yes | End of the query time window |
| **limit** | Integer | No | Maximum records to return. Default: `200`. Hard cap: `200` |
| **entity** | String | No | Data source entity: `netflow` or `lentor`. Default: `netflow` |

#### Flow Record Fields

| Field | Type | Description |
|---|---|---|
| **ts** | String (`YYYY-MM-DD HH:MM:SS`) | Flow start timestamp, newest first |
| **src_ip** | String | Source IP address |
| **dst_ip** | String | Destination IP address |
| **dst_fqdn** | String | Destination hostname from reverse DNS; empty string if unresolved |
| **src_port** | String | Source port number |
| **dst_port** | String | Destination port number |
| **proto** | String | Protocol name: `TCP`, `UDP`, `ICMP`, `ICMPv6`, `GRE`, `ESP`, `VRRP` |
| **flags** | String | TCP flag string as reported by nfdump (e.g. `"...AP.."` = ACK+PSH, `"...R.."` = RST) |
| **packets** | String | Packet count for this flow |
| **bytes** | String | Byte count for this flow |
| **duration** | String | Flow duration in seconds (e.g. `"1.245"`) |
| **verdict** | String | `"Allow"` or `"Deny"` — verdict is `"Deny"` when RST flag is set and SYN is absent |

#### Example: Get Flow Records for a Specific Hour

```
GET /mbox/api/netflow/flows?start=2026/03/25%2009:00&end=2026/03/25%2009:59&limit=50&entity=netflow
Authorization: Bearer {your_token}
```

```json
{
  "status": true,
  "result": {
    "flows": [
      {
        "ts":       "2026-03-25 09:59:45",
        "src_ip":   "10.65.30.2",
        "dst_ip":   "8.8.8.8",
        "dst_fqdn": "dns.google.com",
        "src_port": "45123",
        "dst_port": "53",
        "proto":    "UDP",
        "flags":    "......",
        "packets":  "2",
        "bytes":    "128",
        "duration": "0.120",
        "verdict":  "Allow"
      }
    ],
    "total": 50
  }
}
```

#### Example: Isolating Denied Flows

The API does not natively filter by verdict. To retrieve only denied (rejected) connections, request the full result set and filter client-side:

```
GET /mbox/api/netflow/flows?start=2026/03/25%2000:00&end=2026/03/25%2023:59&limit=200
Authorization: Bearer {your_token}
```

Then filter the response array client-side on `verdict === "Deny"`.

!!! tip
    `denied_flows` in the KPI endpoint gives you the total denied count for a period. Use the flows endpoint to drill into the individual records when investigating a specific incident.

---

### Resolve IP Addresses to FQDN

Performs a batch reverse DNS (PTR record) lookup for up to 20 IP addresses in a single request. Returns a JSON object mapping each queried IP to its resolved hostname. Use this to enrich IP addresses returned in flow records or KPI data with human-readable domain names in external reporting tools.

**Base URL:** `https://YOUR_SERVER_ADDRESS/mbox/api/netflow/resolve`

!!! note
    The request is silently capped at **20 IPs per call**. Invalid IP addresses are skipped. Sending no IPs returns an empty result object.

#### Request Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| **ips** | String | No | Comma-separated list of IP addresses to resolve. Maximum 20 IPs per request |

#### Example Request

```
GET /mbox/api/netflow/resolve?ips=8.8.8.8,1.1.1.1,192.168.1.1
Authorization: Bearer {your_token}
```

#### Example Response

```json
{
  "status": true,
  "result": {
    "8.8.8.8":     "dns.google.com",
    "1.1.1.1":     "one.one.one.one",
    "192.168.1.1": ""
  }
}
```

Each key in `result` is a queried IP address. The value is the resolved FQDN, or an empty string (`""`) if no PTR record exists.

---

## Appendix: Protocol Number Reference

The `protocols` array in the KPI response uses standard IANA protocol numbers:

| Number | Name | Description |
|---|---|---|
| 1 | ICMP | Internet Control Message Protocol |
| 6 | TCP | Transmission Control Protocol |
| 17 | UDP | User Datagram Protocol |
| 47 | GRE | Generic Routing Encapsulation |
| 50 | ESP | Encapsulating Security Payload (IPsec) |
| 58 | ICMPv6 | Internet Control Message Protocol for IPv6 |
| 112 | VRRP | Virtual Router Redundancy Protocol |
