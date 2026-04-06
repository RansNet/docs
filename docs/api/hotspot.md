# HotSpot API

The RansNet Client API v3.1.0 provides secure, real-time access to HotSpot data from the mFusion system. It supports retrieval of historical sessions, active online sessions, login attempt logs, and quota purchase reports — as well as full CRUD management of local users, profiles, and per-user RADIUS access attributes.

---

## Authentication Setup

Before using the HotSpot API, you must enable access through the mFusion GUI:

1. Navigate to **Admin > General > mFusion API** tab.
2. Use the **Entity Search Box** to locate the desired entity, then enable API access by toggling the switch under the **Radius API** column.
3. Click the **Users** icon beside the toggle to manage user access.
4. Find the desired user in the listing and enable the **Radius API** toggle for that user.
5. Click the **key icon** beside the toggle to retrieve the user's **API ID** and **Secret Key**.
6. To generate new credentials, click **Request New Access ID and Secret**.

!!! note
    For full login and authentication details, refer to Chapter 2.1 of the RansNet CLIENT API v3.1.0 Developer Reference Manual.

---

## Methods

| Method | Endpoint | Action |
|--------|----------|--------|
| `POST` | `.../mbox/api/hotspot/auth` | Request JWT token |
| `GET` | `.../mbox/api/hotspot/historical-sessions` | Get all historical session records |
| `GET` | `.../mbox/api/hotspot/online-sessions` | Get all currently connected users |
| `GET` | `.../mbox/api/hotspot/login-attempts` | Get all authentication log data |
| `GET` | `.../mbox/api/hotspot/quota-purchase-report` | Get all user quota purchase report data |
| `GET` | `.../mbox/api/hotspot/users` | Get all local hotspot users |
| `GET` | `.../mbox/api/hotspot/users/{userId}` | Get detailed record for one user |
| `POST` | `.../mbox/api/hotspot/users/{userId}` | Update user details |
| `POST` | `.../mbox/api/hotspot/users` | Create new local user |
| `POST` | `.../mbox/api/hotspot/users/{userId}/actions/attach-profile` | Attach profile to a user |
| `POST` | `.../mbox/api/hotspot/users/{userId}/actions/detach-profile` | Detach profile from a user |
| `DELETE` | `.../mbox/api/hotspot/users/{userId}` | Delete local user |
| `GET` | `.../mbox/api/hotspot/profiles` | Get list of all available profiles |
| `GET` | `.../mbox/api/hotspot/profiles/{profileName}/users` | Get users attached to a profile |
| `GET` | `.../mbox/api/hotspot/user-access` | Get list of all available access attributes |
| `GET` | `.../mbox/api/hotspot/user-access/{userId}` | Get access attributes set for a user |
| `POST` | `.../mbox/api/hotspot/user-access/{userId}/actions/attach-access` | Attach/update access attributes for a user |
| `POST` | `.../mbox/api/hotspot/user-access/{userId}/actions/detach-access` | Detach access attributes from a user |

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

## 3.1 Historical Sessions

Retrieves historical session records for HotSpot users.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/historical-sessions
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `radacctid` | Integer | Record ID |
| `hotspot` | String | NAS ID |
| `username` | String | Authenticated username |
| `nasipaddress` | String (e.g. `192.168.8.1`) | IP address of NAS |
| `macaddress` | String (e.g. `00-60-E0-A3-59-6A`) | User MAC address |
| `calledstationid` | String (e.g. `00-60-E0-A3-59-6A`) | mBox MAC address |
| `framedipaddress` | String (e.g. `192.168.8.1`) | User assigned IP |
| `acctsessiontime` | Integer | Duration of session |
| `acctinputoctets` | Integer | Total bytes received by the user |
| `acctoutputoctets` | Integer | Total bytes sent by the user |
| `acctterminatecause` | String | Reason the session ended |

### Query Parameters

#### Filters

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | String | Filter by authenticated username |
| `hotspot` | String | Filter by NAS ID |
| `nas` | String (e.g. `192.168.8.1`) | Filter by NAS IP address |
| `macaddress` | String (e.g. `00-60-E0-A3-59-6A`) | Filter by customer MAC address |
| `calledstationid` | String (e.g. `00-60-E0-A3-59-6A`) | Filter by mBox MAC address |
| `ipaddress` | String (e.g. `192.168.8.1`) | Filter by user assigned IP |

#### Time Range

| Parameter | Type | Description |
|-----------|------|-------------|
| `startTime` | String (`YYYY-MM-DD` or timestamp) | Start of the time range |
| `endTime` | String (`YYYY-MM-DD` or timestamp) | End of the time range |

#### Pagination

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | Integer | Page number (starts from `1`) |
| `perPage` | Integer | Records per page (default: `100`) |

### Example — Getting Historical Sessions for One Day

```http
GET /mbox/api/hotspot/historical-sessions?startTime=2025-07-21&endTime=2025-07-21&perPage=100
Authorization: Bearer {your_token}
```

**Successful Response (First Page):**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": false,
  "message": "More records available. Use pagination to load next page.",
  "result": [
    {
      "radacctid": 89582,
      "hotspot": "RansNet_vlan210_59-6a",
      "username": "demo_user",
      "macaddress": "3A-71-B4-E9-F7-99",
      "ipaddress": "10.60.208.198",
      "acctstarttime": "2025-07-22 08:33:50",
      "acctstoptime": null,
      "acctsessiontime": 0,
      "calledstationid": "00-60-E0-A3-59-6A",
      "acctinputoctets": 0,
      "acctoutputoctets": 0,
      "acctterminatecause": "",
      "nas": "10.60.208.1"
    }
  ]
}
```

**Successful Response (Last Page):**
```json
{
  "status": true,
  "page": 837,
  "perPage": 100,
  "isLastPage": true,
  "message": "You've reached the last page of the results.",
  "result": [
    {
      "radacctid": 5982,
      "hotspot": "RansNet_vlan210_59-6a",
      "username": "demo_user",
      "macaddress": "4A-71-B4-E9-F7-98",
      "ipaddress": "10.60.211.19",
      "acctstarttime": "2025-06-07 08:29:26",
      "acctstoptime": "2025-06-07 09:19:34",
      "acctsessiontime": 3008,
      "calledstationid": "00-60-E0-A3-59-6A",
      "acctinputoctets": 96042755,
      "acctoutputoctets": 8685308,
      "acctterminatecause": "Idle-Timeout",
      "nas": "10.60.210.1"
    }
  ]
}
```

### Pagination Handling

Check the `isLastPage` field to determine if more pages are available. Increment `page` while `isLastPage` is `false`.

!!! warning
    Avoid setting `perPage` too high. Excessively large page sizes may cause memory exhaustion on the mBox. Keep `perPage` at or below `1000`.

---

## 3.2 Online Sessions

Retrieves real-time session data for users currently connected to the HotSpot network.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/online-sessions
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `radacctid` | Integer | Record ID |
| `hotspot` | String | NAS ID |
| `username` | String | Authenticated username |
| `nas` | String (e.g. `192.168.8.1`) | IP address of NAS |
| `macaddress` | String (e.g. `00-60-E0-A3-59-6A`) | User MAC address |
| `calledstationid` | String (e.g. `00-60-E0-A3-59-6A`) | mBox MAC address |
| `ipaddress` | String (e.g. `192.168.8.1`) | User assigned IP address |
| `starttime` | String (datetime) | Session start time |
| `traffic` | String | Total bytes sent and received by the user |
| `name` | String | User first and last name |
| `totaltime` | String | Total session duration |

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | String | Filter by authenticated username |
| `hotspot` | String | Filter by NAS ID |
| `nas` | String (e.g. `192.168.8.1`) | Filter by NAS IP address |
| `macaddress` | String (e.g. `3A-71-B4-E9-F7-99`) | Filter by customer MAC address |
| `calledstationid` | String (e.g. `00-60-E0-A3-59-6A`) | Filter by mBox MAC address |
| `startTime` | String (`YYYY-MM-DD` or timestamp) | Start of the time range |
| `endTime` | String (`YYYY-MM-DD` or timestamp) | End of the time range |
| `perPage` | Integer | Records per page (default: `100`) |
| `page` | Integer | Page number for pagination |

### Example — Get Online Sessions by mBox MAC

```http
GET /mbox/api/hotspot/online-sessions?calledstationid=00-60-E0-A3-59-6A&perPage=100
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": true,
  "message": "You've reached the last page of the results.",
  "result": [
    {
      "username": "demo_user",
      "ipaddress": "10.60.208.198",
      "macaddress": "3A-71-B4-E9-F7-99",
      "starttime": "2025-07-22 08:33:50",
      "totaltime": "0sec",
      "nas": "10.60.208.1",
      "traffic": "0 MB",
      "calledstationid": "00-60-E0-A3-59-6A",
      "radacctid": 89582,
      "hotspot": "RansNet_vlan210_59-6a",
      "name": ""
    }
  ]
}
```

---

## 3.3 Login Attempts

Retrieves authentication logs for HotSpot user logins. Records every login attempt (accepted or rejected) with username, masked password, and timestamp.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/login-attempts
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | String | Filter by account username |
| `reply` | String | Filter by result: `Access-Accept` or `Access-Reject` |
| `startTime` | String (`YYYY-MM-DD` or timestamp) | Start of the time range |
| `endTime` | String (`YYYY-MM-DD` or timestamp) | End of the time range |
| `perPage` | Integer | Records per page (default: `100`) |
| `page` | Integer | Page number for pagination |

### Example — Get Rejected Login Attempts for a User

```http
GET /mbox/api/hotspot/login-attempts?username=demo_user&reply=Access-Reject&perPage=100
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": false,
  "message": "More records available. Use pagination to load next page.",
  "result": [
    {
      "id": 95419,
      "username": "demo_user",
      "pass": "********",
      "reply": "Access-Reject",
      "authdate": "2025-07-22 00:52:40"
    },
    {
      "id": 95417,
      "username": "demo_user",
      "pass": "********",
      "reply": "Access-Reject",
      "authdate": "2025-07-22 00:52:23"
    }
  ]
}
```

---

## 3.4 Quota Purchase Report

Retrieves user quota top-up and purchase history, including plan details, transaction time, and remarks.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/quota-purchase-report
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `transaction_time` | String | Filter by transaction time |
| `username` | String | Filter by account username |
| `plan_name` | String | Filter by plan name |
| `user_profiles` | String | Filter by attached profiles |
| `remarks` | String | Filter by purchase remarks |
| `startTime` | String (`YYYY-MM-DD` or timestamp) | Start of the time range |
| `endTime` | String (`YYYY-MM-DD` or timestamp) | End of the time range |
| `perPage` | Integer | Records per page (default: `100`) |
| `page` | Integer | Page number for pagination |

### Example — Get Quota Purchases for a Month

```http
GET /mbox/api/hotspot/quota-purchase-report?startTime=2025-12-01&endTime=2025-12-31&user_profiles=Vessel-1001&perPage=100
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": true,
  "message": "You've reached the last page of the results.",
  "result": [
    {
      "id": 50,
      "transaction_time": "2025-12-17 13:20:13",
      "username": "username3002",
      "plan_name": "2GB-USD$8.00",
      "currency": "USD",
      "amount": "8.00",
      "user_profiles": "Vessel-1001",
      "remarks": "Manual Topup Quota - this is an added custom remarks",
      "create_by": "mboxadmin",
      "create_at": "2025-12-17 13:20:13"
    },
    {
      "id": 49,
      "transaction_time": "2025-12-17 13:07:21",
      "username": "username3002",
      "plan_name": "4GB-USD$8.00",
      "currency": "USD",
      "amount": "8.00",
      "user_profiles": "Vessel-1001",
      "remarks": "Manual Topup Quota - added remarks",
      "create_by": "mboxadmin",
      "create_at": "2025-12-17 13:07:21"
    }
  ]
}
```

---

## 4.1 Managing Users

Full CRUD operations (Create, Retrieve, Update, Delete) on local HotSpot user accounts.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/users
```

### GET — List Users or Get Single User

| Endpoint | Action |
|----------|--------|
| `/users` | List all users |
| `/users/{userId}` | Get detailed record for one user |

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | Integer | Unique user record ID (for single user retrieval) |
| `username` | String | Filter by account username |
| `page` | Integer | Page number |
| `perPage` | Integer | Records per page (default: `100`) |

**Example — Get Users List:**
```http
GET /mbox/api/hotspot/users
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": false,
  "message": "More records available. Use pagination to load next page.",
  "result": [
    {
      "id": 1889,
      "username": "00-8b-1a-2b-3c-4d-6e",
      "usertype": "Auth-Type",
      "profile": null,
      "firstname": null,
      "lastname": null,
      "email": null,
      "mobile": null,
      "gender": null,
      "disabled": false
    }
  ]
}
```

**Example — Get Single User:**
```http
GET /mbox/api/hotspot/users/1890
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "User details retrieved successfully.",
  "result": [
    {
      "id": 1890,
      "username": "johndoe",
      "firstname": "John",
      "lastname": "Doe",
      "email": "johndoe@gmail.com",
      "mobile": "12345678",
      "gender": "male",
      "createddate": "2025-11-03 09:49:38",
      "createdby": "mboxadmin",
      "updateddate": "2025-11-03 09:50:04",
      "updatedby": "mboxadmin",
      "usertype": "Username-Password",
      "profile": "Data-Max-Daily-10GB, Device-Max-1",
      "password": "S*******",
      "disabled": false
    }
  ]
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "errorMessage": "User with ID: 1893 not found.",
  "result": []
}
```

---

### POST — Create New User

**Endpoint:** `/users`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | String | **Yes** | New user's username |
| `password` | String | **Yes** | New user's password (for `user` type) |
| `usertype` | String | **Yes** | Authentication type: `user` (password), `mac`, or `pin` |
| `firstname` | String | No | User's first name |
| `lastname` | String | No | User's last name |
| `email` | String | No | User's email |
| `mobile` | String | No | User's mobile |
| `gender` | String | No | User's gender |
| `race` | String | No | User's race |
| `religion` | String | No | User's religion |
| `nationality` | String | No | User's nationality |
| `profession` | String | No | User's profession |
| `interests` | String | No | User's interests |
| `marital_status` | String | No | User's marital status |
| `birthdate` | String (`YYYY-MM-DD`) | No | User's birthday |
| `locale` | String | No | User's locale |
| `passport_nric_no` | String | No | User's passport/NRIC number |
| `company` | String | No | User's company |
| `address` | String | No | User's address |
| `country` | String | No | User's country |
| `zip` | String | No | User's zip code |
| `disabled` | Boolean | No | Disable the user account |
| `profiles` | String | No | Single or comma-separated profile names to attach |

**Example:**
```http
POST /mbox/api/hotspot/users
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "usertype": "pin",
  "username": "pin1234"
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "User created successfully.",
  "result": {
    "id": "1893",
    "username": "pin1234",
    "createdby": "RADIUS API",
    "createddate": "2025-11-03 10:59:11",
    "updatedby": "RADIUS API",
    "updateddate": "2025-11-03 10:59:12"
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "errorMessage": "Username 'pin1234' already exists.",
  "result": []
}
```

---

### POST — Update Existing User

**Endpoint:** `/users/{userId}`

| Parameter | Type | Description |
|-----------|------|-------------|
| `username` | String | Username of the user to update |
| `password` | String | Optional: new password |
| `email` | String | Field to update |
| `...` | ... | All other user info fields |

**Example:**
```http
POST /mbox/api/hotspot/users/1891
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "firstname": "Newusername",
  "lastname": "Newlastname"
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "User successfully updated.",
  "result": [
    {
      "id": 1890,
      "username": "johndoe",
      "firstname": "Newusername",
      "lastname": "Newlastname",
      "email": "johndoe@gmail.com",
      "updateddate": "2025-11-03 10:21:18",
      "updatedby": "RADIUS API",
      "usertype": "Username-Password",
      "profile": "Data-Max-Daily-10GB, Device-Max-1",
      "password": "S*******",
      "disabled": false
    }
  ]
}
```

---

### DELETE — Delete User

**Endpoint:** `/users/{userId}`

| Parameter | Type | Description |
|-----------|------|-------------|
| `userId` | Integer | Unique ID of the user to delete |

Removes the user and all associated records from the authentication, group, and profile tables.

**Example:**
```http
DELETE /mbox/api/hotspot/users/1892
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "User 'ransnetuser' (ID: 1892) successfully deleted.",
  "result": {
    "id": 1892,
    "username": "ransnetuser"
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "errorMessage": "User with ID: 1892 not found.",
  "result": []
}
```

---

### POST — Attach Profile to User

**Endpoint:** `/users/{userId}/actions/attach-profile`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profileName` | String | **Yes** | Name of the profile to attach |

**Example:**
```http
POST /mbox/api/hotspot/users/1890/actions/attach-profile
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "profileName": "Add_on_1_device"
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "Profile 'Add_on_1_device' successfully attached to user 'johndoe'.",
  "result": {
    "username": "johndoe",
    "attached": "Add_on_1_device",
    "profile": "Data-Max-Daily-10GB, Device-Max-1, Add_on_1_device"
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "errorMessage": "User 'johndoe' is already attached to profile: 'Add_on_1_device'.",
  "result": []
}
```

---

### POST — Detach Profile from User

**Endpoint:** `/users/{userId}/actions/detach-profile`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profileName` | String | **Yes** | Name of the profile to detach |

**Example:**
```http
POST /mbox/api/hotspot/users/1890/actions/detach-profile
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "profileName": "Add_on_1_device"
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "Profile 'Add_on_1_device' successfully detached from user 'johndoe'.",
  "result": {
    "username": "johndoe",
    "detached": "Add_on_1_device",
    "profile": "Data-Max-Daily-10GB, Device-Max-1"
  }
}
```

**Sample Error Response:**
```json
{
  "status": true,
  "message": "User 'johndoe' was not attached to profile 'Add_on_1_device'. No action needed.",
  "result": []
}
```

---

## 4.2 Managing Profiles

Provides access to the defined HotSpot access profiles stored in the database.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/profiles
```

!!! note
    `profileName` must be alphanumeric and may include `-` and `_` characters.

### GET — List All Profiles

**Endpoint:** `/profiles`

Returns `groupname` and `description` for each profile.

**Example:**
```http
GET /mbox/api/hotspot/profiles
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": false,
  "message": "More records available. Use pagination to load next page.",
  "result": [
    {
      "groupname": "1902b",
      "description": "1902b"
    },
    {
      "groupname": "2115",
      "description": "2115"
    }
  ]
}
```

### GET — List Users by Profile

**Endpoint:** `/profiles/{profileName}/users`

Returns all user records linked to the specified profile.

**Example:**
```http
GET /mbox/api/hotspot/profiles/Add_on_1_device/users
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "page": 1,
  "perPage": 100,
  "isLastPage": true,
  "message": "You've reached the last page of the results.",
  "result": [
    {
      "id": "1883",
      "username": "jasontest",
      "firstname": "John",
      "lastname": "Doe",
      "profileName": "Add_on_1_device",
      "priority": "1"
    }
  ]
}
```

---

## 4.3 Managing User Access

Manages fine-grained RADIUS attributes set on individual users, which override profile-level settings.

**Base URL:**
```
https://YOUR_SERVER_ADDRESS/mbox/api/hotspot/user-access
```

!!! note
    `userId` must be a valid Integer.

### GET — List All Available Attributes

**Endpoint:** `/user-access`

Returns all available attribute definitions (e.g., `Session-Timeout`, `CS-Total-MB`) with their configuration (text, format).

**Example:**
```http
GET /mbox/api/hotspot/user-access
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "User Attributes retrieved successfully.",
  "result": {
    "WISPr-Bandwidth-Max-Up": {
      "text": "Maximum Upload",
      "format": "Kbps",
      "multiple": false
    },
    "WISPr-Bandwidth-Max-Down": {
      "text": "Maximum Download",
      "format": "Kbps",
      "multiple": false
    }
  }
}
```

### GET — Get Attributes for a User

**Endpoint:** `/user-access/{userId}`

Retrieves all active attributes and their current values set on the specified user.

**Example:**
```http
GET /mbox/api/hotspot/user-access/1890
Authorization: Bearer {your_token}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "Access attributes for user retrieved successfully.",
  "result": {
    "id": 1890,
    "username": "johndoe",
    "attributes": [
      {
        "Simultaneous-Use": {
          "id": "3266",
          "format": "Number",
          "value": 1
        }
      },
      {
        "WISPr-Bandwidth-Max-Up": {
          "id": "140",
          "format": "Kbps",
          "value": 5000
        }
      }
    ]
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "errorMessage": "No user found.",
  "result": []
}
```

---

### POST — Attach / Update Attributes

**Endpoint:** `/user-access/{userId}/actions/attach-access`

Adds a new attribute or updates the value of an existing attribute for a user. Supports single or bulk operations.

**Single Attribute Input:**

| Input | Type | Description |
|-------|------|-------------|
| `attribute` | String | Attribute name |
| `value` | String | Value to set |

**Bulk Attribute Input:**

| Input | Type | Description |
|-------|------|-------------|
| `attributes` | Array | Array of objects: `[{"attribute": "Name", "value": "XXX"}]` |

**Example — Attach Single Attribute:**
```http
POST /mbox/api/hotspot/user-access/1890/actions/attach-access
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "attribute": "CS-Total-MB-Daily",
  "value": "40"
}
```

**Example — Attach Multiple Attributes:**
```http
POST /mbox/api/hotspot/user-access/1890/actions/attach-access
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "attributes": [
    {"attribute": "CS-Total-MB-Daily", "value": "40"},
    {"attribute": "WISPr-Redirection-URL", "value": "https://google.com"}
  ]
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "Processed user access 2 success. 0 failed.",
  "details": {
    "success": [
      {
        "attribute": "CS-Total-MB-Daily",
        "value": 40,
        "action": "User acess 'CS-Total-MB-Daily' successfully attached."
      },
      {
        "attribute": "WISPr-Redirection-URL",
        "value": "https://google.com",
        "action": "User acess 'WISPr-Redirection-URL' successfully attached."
      }
    ]
  },
  "result": {
    "id": 1890,
    "username": "johndoe",
    "user_access": {
      "CS-Total-MB-Daily": {"id": "3270", "format": "MB", "value": "40"},
      "WISPr-Redirection-URL": {"id": "141", "format": "URL", "value": "https://google.com"}
    }
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "message": "Processed user access 0 success. 1 failed.",
  "details": {
    "errors": ["User access 'CS-Total-MB-Daily22222' is not recognized."]
  }
}
```

---

### POST — Detach Attributes

**Endpoint:** `/user-access/{userId}/actions/detach-access`

Removes an attribute from a user's access settings. Supports single or bulk operations.

**Single Attribute Input:**

| Input | Type | Description |
|-------|------|-------------|
| `attribute` | String | Attribute name to delete |

**Bulk Attribute Input:**

| Input | Type | Description |
|-------|------|-------------|
| `attributes` | Array | Array of objects: `[{"attribute": "Name"}]` |

**Example:**
```http
POST /mbox/api/hotspot/user-access/1890/actions/detach-access
Authorization: Bearer {your_token}
Content-Type: application/json

{
  "attribute": "CS-Total-MB-Daily"
}
```

**Successful Response:**
```json
{
  "status": true,
  "message": "Processed user access 1 success. (and skipped 0 existing). 0 failed.",
  "details": {
    "success": [
      {
        "attribute": "CS-Total-MB-Daily",
        "action": "detached",
        "message": "User access 'CS-Total-MB-Daily' successfully detached."
      }
    ]
  },
  "result": {
    "id": 1890,
    "username": "johndoe"
  }
}
```

**Sample Error Response:**
```json
{
  "status": false,
  "message": "Processed user access 0 success. (and skipped 0 existing). 1 failed.",
  "details": {
    "errors": ["User access 'CS-Total-MB-Daily' was already detached or never existed."]
  }
}
```
