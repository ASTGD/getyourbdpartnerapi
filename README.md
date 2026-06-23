# GetYour Partner API

Base URL: `https://getyour.com.bd`

## Authentication

All endpoints use **HTTP Basic Authentication**.

Use your assigned:

* **User ID** as the username
* **Password** as the password

Example:

```bash
curl -u "USER_ID:PASSWORD"
```

---

# Domain Order API

Create a new domain order and generate an invoice for payment.

## Endpoint

```http
POST /api/v1/domain/orders
```

### Request

* Content-Type: `multipart/form-data`
* Rate limit: `60 requests / minute`
* Authentication: Required (Basic Auth)

### Required Fields

| Field          | Type   | Description                   |
| -------------- | ------ | ----------------------------- |
| domain         | string | Domain name to register       |
| nameServers[]  | array  | 2–3 name servers              |
| fullName       | string | Registrant full name          |
| nid            | string | National ID (10/13/17 digits) |
| nid_document   | file   | JPG, JPEG, PNG, or PDF        |
| email          | string | Valid email address           |
| contactAddress | string | Contact address               |
| contactNumber  | string | Format: +880XXXXXXXXXX        |

### Optional Fields

| Field                 | Type    | Description                    |
| --------------------- | ------- | ------------------------------ |
| years                 | integer | Registration term (1–10 years) |
| registration_document | file    | Required for some TLDs         |

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

### Request

* Content-Type: `application/json`
* Rate limit: `60 requests / minute`
* Authentication: Required (Basic Auth)

### Request Body

| Field       | Type          | Required | Description      |
| ----------- | ------------- | -------- | ---------------- |
| domain      | string        | Yes      | Full domain name |
| nameServers | array[string] | Yes      | 2–3 name servers |

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

| HTTP Code | Description            |
| --------- | ---------------------- |
| 401       | Unauthenticated        |
| 404       | Domain order not found |
| 422       | Validation failed      |
| 422       | Domain suspended       |
| 502       | Registrar sync failed  |

### Notes

* Send the full domain name in the `domain` field.
* Use `nameServers`, not `name_servers`.
* Minimum name servers: 2.
* Maximum name servers: 3.
* Suspended domains cannot be updated.
* Pending domains are updated locally only.
* Active domains are updated locally and synchronized with the registrar.
