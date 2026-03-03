# GetYour Partner API

Base URL: `https://getyour.com.bd`

## 📚 Index
- [Domain Availability API](#domain-availability-api)
- [Domain Order API](#domain-order-api)

---

## Domain Availability API
Checks whether a domain is available for registration.

### Endpoint
`POST /api/v1/domain/availability`

### Headers
- `Content-Type: application/json`
- `x-api-key: YOUR_PARTNER_API_KEY`

### Request Body
```json
{
  "domain": "example.bd"
}
```

### Example Request (cURL)
```bash
curl -X POST "https://your-api-domain.com/api/v1/domain/availability" \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: YOUR_PARTNER_API_KEY" \\
  -d '{"domain":"example.bd"}'
```

### Success Response (HTTP 200)
```json
{
  "responseCode": 2000,
  "status": "success",
  "message": "Operation successful"
}
```

### Error Responses

#### Validation Error (HTTP 400)
```json
{
  "message": "Validation failed",
  "details": {
    "formErrors": [],
    "fieldErrors": {
      "domain": ["Enter a valid domain name"]
    }
  }
}
```

#### Missing API Key (HTTP 401)
```json
{
  "message": "API key required"
}
```

#### Invalid API Key (HTTP 401)
```json
{
  "message": "Invalid API key"
}
```

#### Rate Limit Exceeded (HTTP 429)
```json
{
  "message": "Rate limit exceeded"
}
```

#### Upstream Business Error Examples
```json
{
  "responseCode": 2001,
  "status": "error",
  "message": "Domain already registered"
}
```

```json
{
  "responseCode": 2005,
  "status": "error",
  "message": "Domain reserved by others"
}
```

```json
{
  "responseCode": 4002,
  "status": "error",
  "message": "Invalid domain format"
}
```

#### Internal Server Error (HTTP 500)
```json
{
  "status": "error",
  "responseCode": 5000,
  "message": "Internal server error"
}
```

### Common `responseCode` values
- `2000` = Operation successful
- `2001` = Domain already registered
- `2004` = Domain already reserved by you
- `2005` = Domain reserved by others
- `4002` = Invalid domain format
- `4015` = Invalid upstream API credentials
- `4016` = Account inactive
- `4030` = Reseller not allowed
- `5000` = Internal server error

---

## Domain Order API
Creates a domain order and generates an invoice.

### Authentication
This API uses **HTTP Basic Auth**.
Use your **User ID** as username and **password** as password.

### Endpoint
`POST /api/v1/domain/orders`

### Request format
- Content-Type: `multipart/form-data`
- Rate limit: `60 requests / minute`
- Auth required: Yes (Basic Auth)

### Required fields
- `domain` (string)
- `nameServers[0]` (string)
- `nameServers[1]` (string)
- `fullName` (string)
- `nid` (string; 10/13/17 digits)
- `nid_document` (file: jpg/jpeg/png/pdf)
- `email` (valid email)
- `contactAddress` (string)
- `contactNumber` (`+880` followed by 10 digits)

### Optional fields
- `nameServers[2]` (string)
- `years` (1–10)
- `registration_document` (file; required for some TLDs)

### Success response (HTTP 201)
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
