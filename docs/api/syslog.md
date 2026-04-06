# Syslog / Logviewer API

The RansNet Client API v3.1.0 provides secure, real-time access to syslog data from the mFusion system. It supports log retrieval by time range, host, message content, tag, and priority level — designed for system integrators and developers building external log management or monitoring solutions.

---

## Authentication Setup

Before using the Syslog API, you must enable access through the mFusion GUI:

1. Navigate to **Admin > General > mFusion API** tab.
2. Use the **Entity Search Box** to locate the desired entity, then enable API access by toggling the switch under the **Logviewer API** column.
3. Click the **Users** icon beside the toggle to manage user access.
4. Find the desired user in the listing and enable the **Logviewer API** toggle for that user.
5. Click the **key icon** beside the toggle to retrieve the user's **API ID** and **Secret Key**.
6. To generate new credentials, click **Request New Access ID and Secret**.

!!! note
    For full login and authentication details, refer to Chapter 2.1 of the RansNet CLIENT API v3.1.0 Developer Reference Manual.

---

## Methods

| Method | Endpoint | Action | Response |
|--------|----------|--------|----------|
| `POST` | `.../mbox/api/logviewer/auth` | Request JWT token | JSON object |
| `GET` | `.../mbox/api/logviewer/search` | Search syslog records with filters | JSON object |
| `GET` | `.../mbox/api/logviewer/latest` | Get latest syslog records | JSON object |

---

## Responses

All API responses are JSON objects with a `status` property indicating success or failure.

### Successful Response (`status: true`)

| Field | Type | Description |
|-------|------|-------------|
| `page` | Integer | Current page number of the result set |
| `perPage` | Integer | Page size of records |
| `isLastPage` | Boolean | `false` = more data available; `true` = last page |
| `message` | String | Optional message describing the result |
| `result` | Array | Data items matching the request |

### Failed Response (`status: false`)

| Field | Description |
|-------|-------------|
| `errorMessage` | Reason for failure (e.g., `Token validation failed`) |

---

## 3.1 Search Syslog Logs

Retrieves syslog records from the `SystemEvents` table with optional filtering by host, message content, tag, priority, and time range.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/logviewer/search
```

### Response Fields

Each record in the `result` array contains:

| Field | Type | Description |
|-------|------|-------------|
| `ID` | Integer | Record ID |
| `DeviceReportedTime` | String (datetime) | Timestamp when device reported the event |
| `ReceivedAt` | String (datetime) | Server timestamp when log was received |
| `FromHost` | String | Source hostname or IP address |
| `SysLogTag` | String | Syslog program/tag identifier |
| `processid` | String | Process ID from syslog |
| `Message` | String | Full log message content |
| `Priority` | Integer (0–7) | Syslog priority level |

### Query Parameters

#### Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `host` | String | Filter by source hostname or IP address (exact match) |
| `message` | String | Filter by message content (substring match, `LIKE %value%`) |
| `tag` | String | Filter by syslog tag / program name (exact match) |
| `priority` | Integer (0–7) | Filter by syslog priority level (exact match) |

#### Time Range

| Parameter | Type | Description |
|-----------|------|-------------|
| `startTime` | String (`YYYY-MM-DD` or timestamp) | Start of the time range for the log search |
| `endTime` | String (`YYYY-MM-DD` or timestamp) | End of the time range for the log search |

#### Pagination

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | Integer | Page number (starts from `1`) |
| `perPage` | Integer | Records per page (default: `100`, max: `2000`) |

### Examples

#### Example 1 — Search by Host

```http
GET /mbox/api/logviewer/search?host=10.18.18.178&perPage=20
Authorization: Bearer {your_token}
```

**Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 20,
  "isLastPage": false,
  "message": "More records available. Use pagination to load next page.",
  "result": [
    {
      "ID": 2324827,
      "DeviceReportedTime": "2025-07-05 12:02:44",
      "ReceivedAt": "2025-07-05 20:02:44",
      "FromHost": "10.18.18.178",
      "SysLogTag": "agent-watchdog:",
      "processid": "",
      "Message": " ezagent ath1 connected",
      "Priority": 5
    }
  ]
}
```

#### Example 2 — Search by Time Range and Message Keyword

```http
GET /mbox/api/logviewer/search?startTime=2025-07-05&endTime=2025-07-06&message=disassociated&perPage=50
Authorization: Bearer {your_token}
```

#### Example 3 — Search by Priority Level

```http
GET /mbox/api/logviewer/search?priority=6&perPage=20
Authorization: Bearer {your_token}
```

#### Example 4 — Search by Syslog Tag

```http
GET /mbox/api/logviewer/search?tag=hostapd:&perPage=20
Authorization: Bearer {your_token}
```

### Pagination Handling

The API supports pagination for large datasets. Use `page` and `perPage` to navigate results.

- Check the `isLastPage` field to determine if more pages are available.
- When `isLastPage` is `false`, increment `page` to fetch the next set of records.
- When `isLastPage` is `true`, all records have been retrieved.

!!! warning
    Avoid setting `perPage` too high. Excessively large page sizes may cause memory exhaustion or performance degradation on the mBox. Keep `perPage` at or below `1000` depending on expected row size.

---

## 3.2 Get Latest Syslog Logs

Retrieves the most recent syslog entries ordered by most recent first. Useful for quickly checking current log activity without specifying a time range.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/logviewer/latest
```

!!! note
    This endpoint does **not** support `startTime`/`endTime` parameters. Use the [search endpoint](#31-search-syslog-logs) for time range filtering.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `host` | String | Filter by source hostname or IP address (exact match) |
| `message` | String | Filter by message content (substring match) |
| `tag` | String | Filter by syslog tag / program name (exact match) |
| `priority` | Integer (0–7) | Filter by syslog priority level (exact match) |
| `perPage` | Integer | Records per page (default: `100`, max: `2000`) |
| `page` | Integer | Page number for pagination |

### Examples

#### Example 1 — Get Latest 20 Entries

```http
GET /mbox/api/logviewer/latest?perPage=20
Authorization: Bearer {your_token}
```

#### Example 2 — Get Latest Logs from a Specific Host

```http
GET /mbox/api/logviewer/latest?host=10.18.18.39&perPage=50
Authorization: Bearer {your_token}
```

#### Example 3 — Get Latest Logs by Tag

```http
GET /mbox/api/logviewer/latest?tag=agent-watchdog:&perPage=20
Authorization: Bearer {your_token}
```

---

## Appendix: Syslog Priority Levels

| Priority | Severity | Description |
|----------|----------|-------------|
| `0` | Emergency | System is unusable |
| `1` | Alert | Action must be taken immediately |
| `2` | Critical | Critical conditions |
| `3` | Error | Error conditions |
| `4` | Warning | Warning conditions |
| `5` | Notice | Normal but significant condition |
| `6` | Informational | Informational messages |
| `7` | Debug | Debug-level messages |
