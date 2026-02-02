# GetYour Partner API — Domain Order

Base URL: `https://getyour.com.bd`

## Authentication
This API uses **HTTP Basic Auth**.
Use your **User ID** as username and **password** as password.

## Endpoint
`POST /api/v1/domain/orders`

- Content-Type: `multipart/form-data`
- Rate limit: `60 requests / minute`
- Auth required: Yes (Basic Auth)

## Required fields
- `domain` (string)
- `nameServers[]` (2–3 strings; indexes 0 and 1 required)
- `fullName` (string)
- `nid` (string; 10/13/17 digits)
- `nid_document` (file: jpg/jpeg/png/pdf)
- `email` (valid email)
- `contactAddress` (string)
- `contactNumber` (+880 followed by 10 digits)

Optional:
- `years` (1–10)
- `registration_document` (file; required for some TLDs)

## Success response (201)
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
