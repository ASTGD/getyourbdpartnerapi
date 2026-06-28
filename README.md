# GetYour Partner API

**Base URL**

```text
https://getyour.com.bd
```

---

# Authentication

The API uses two authentication methods depending on the endpoint.

## HTTP Basic Authentication

The following endpoints require **HTTP Basic Authentication**:

* Domain Order API
* Update Name Servers API

Use your assigned:

* **User ID** as the username
* **Password** as the password

Example:

```bash
curl -u "USER_ID:PASSWORD"
```

---

## API Key Authentication

The **Domain Availability API** uses an API key.

Send your API key in the `x-api-key` request header.

```http
x-api-key: YOUR_API_KEY
```

Supported API keys:

* `FULL_ACCESS`
* `SEARCH_ONLY`

---

# Domain Availability API

Check whether a domain is available for registration.

## Endpoint

```http
POST /api/v1/domain/availability
```

### Request

* Content-Type: `application/json`
* Authentication: API Key (`x-api-key`)
* Supported Keys: `FULL_ACCESS`, `SEARCH_ONLY`

### Request Headers

| Header         | Required | Description                |
| -------------- | -------- | -------------------------- |
| `Content-Type` | Yes      | Must be `application/json` |
| `x-api-key`    | Yes      | Your partner API key       |

### Request Body

```json
{
  "domain": "example.com"
}
```

### Parameters

| Field    | Type   | Required | Description                    |
| -------- | ------ | -------- | ------------------------------ |
| `domain` | string | Yes      | Full domain name including TLD |

### Example Request

```bash
curl -X POST https://domainapi.astgd.com/api/v1/domain/availability \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"domain":"example.com"}'
```

### Success Response

**HTTP 200**

```json
{
  "domain": "example.com",
  "status": "success",
  "responseCode": 2000,
  "message": "Operation successful"
}
```

### Error Responses

#### Missing API Key

**HTTP 401**

```json
{
  "message": "API key required"
}
```

#### Invalid API Key

**HTTP 401**

```json
{
  "message": "Invalid API key"
}
```

#### Invalid Request Body

**HTTP 400**

```json
{
  "message": "Validation failed",
  "details": [
    {
      "path": ["domain"],
      "message": "Required"
    }
  ]
}
```

### HTTP Status Codes

| Status Code | Description                                               |
| ----------- | --------------------------------------------------------- |
| `200`       | Request processed successfully                            |
| `400`       | Invalid request body                                      |
| `401`       | Missing or invalid API key                                |
| `403`       | Forbidden (SEARCH_ONLY key used on unauthorized endpoint) |
| `429`       | Rate limit exceeded                                       |
| `500`       | Internal server error                                     |

### Notes

* The `domain` field must contain the full domain name including its extension (e.g. `example.com`).
* API keys are rate-limited.
* Both `FULL_ACCESS` and `SEARCH_ONLY` API keys can access this endpoint.
* `SEARCH_ONLY` API keys cannot access any other `/api/v1/*` endpoints. Attempting to do so returns **HTTP 403 Forbidden**.

---

# Domain Order API

Create a new domain order and generate an invoice for payment.

## Endpoint

```http
POST /api/v1/domain/orders
```

## Request

* Content-Type: `multipart/form-data`
* Authentication: HTTP Basic Authentication
* Rate limit: **60 requests/minute**

### Required Fields

| Field            | Type   | Description                        |
| ---------------- | ------ | ---------------------------------- |
| `domain`         | string | Domain name to register            |
| `nameServers[]`  | array  | 2–3 name servers                   |
| `fullName`       | string | Registrant full name               |
| `nid`            | string | National ID (10, 13, or 17 digits) |
| `nid_document`   | file   | JPG, JPEG, PNG, or PDF             |
| `email`          | string | Valid email address                |
| `contactAddress` | string | Contact address                    |
| `contactNumber`  | string | Format: `+880XXXXXXXXXX`           |

### Optional Fields

| Field                   | Type    | Description                    |
| ----------------------- | ------- | ------------------------------ |
| `years`                 | integer | Registration term (1–10 years) |
| `registration_document` | file    | Required for some TLDs         |

### Success Response

**HTTP 201**

```json
{
  "message": "Domain order created. Invoice generated and pending payment.",
  "order": {
    "id": 123,
    "domain": "example.com",
    "status": "pending",
    "registration_status": "pending",
    "term_years": 1
  },
  "invoice": {
    "id": 456,
    "status": "unpaid",
    "payment_method": "api",
    "total": 1234.56
  },
  "pricing": {
    "subtotal": "...",
    "vat": "...",
    "platform_fee": "...",
    "total": "..."
  }
}
```

---

# Update Name Servers API

Update the name servers of an existing domain order.

## Endpoint

```http
POST /api/v1/domain/update-ns
```

## Request

* Content-Type: `application/json`
* Authentication: HTTP Basic Authentication
* Rate limit: **60 requests/minute**

### Request Body

| Field         | Type          | Required | Description      |
| ------------- | ------------- | -------- | ---------------- |
| `domain`      | string        | Yes      | Full domain name |
| `nameServers` | array[string] | Yes      | 2–3 name servers |

### Example Request

```json
{
  "domain": "example.com.bd",
  "nameServers": [
    "ns1.example.com",
    "ns2.example.com"
  ]
}
```

### Success Response (Pending Domain)

```json
{
  "message": "Name servers updated successfully.",
  "order": {
    "id": 123,
    "domain": "example.com.bd",
    "status": "pending",
    "nameServers": [
      "ns1.example.com",
      "ns2.example.com"
    ]
  },
  "registrar": null
}
```

### Success Response (Active Domain)

```json
{
  "message": "Name servers updated successfully.",
  "order": {
    "id": 123,
    "domain": "example.com.bd",
    "status": "active",
    "nameServers": [
      "ns1.example.com",
      "ns2.example.com"
    ]
  },
  "registrar": {
    "success": true,
    "payload": {},
    "message": "Name servers updated successfully."
  }
}
```

### Error Responses

| HTTP Code | Description                      |
| --------- | -------------------------------- |
| `401`     | Unauthenticated                  |
| `404`     | Domain order not found           |
| `422`     | Validation failed                |
| `422`     | Domain suspended                 |
| `502`     | Registrar synchronization failed |

### Notes

* Send the complete domain name in the `domain` field.
* Use `nameServers` (camelCase), **not** `name_servers`.
* Minimum supported name servers: **2**.
* Maximum supported name servers: **3**.
* Suspended domains cannot be updated.
* Pending domains are updated locally only.
* Active domains are updated locally and synchronized with the registrar.
