# Errors & Troubleshooting

Coinspe returns standard HTTP status codes along with JSON bodies that describe what went wrong. Use the table below as a quick decoder ring when handling responses from the payment gateway.

## Status catalog

| HTTP Status | Name | Description |
| --- | --- | --- |
| `200 OK` | Success | Request processed successfully. |
| `201 Created` | Created | Resource (payment link) created successfully. |
| `400 Bad Request` | Bad Request | Malformed request or missing required headers/parameters. |
| `401 Unauthorized` | Unauthorized | Invalid/missing bearer token or invalid HMAC signature. |
| `404 Not Found` | Not Found | Payment link token does not exist. |
| `422 Unprocessable Entity` | Validation failed | Check the `errors` object for field-level issues. |
| `500 Internal Server Error` | Server error | Coinspe issue; retry with backoff and contact support if persistent. |

### Standard error envelope

```json
{
  "message": "Error description.",
  "errors": {
    "field_name": [
      "Validation message."
    ]
  }
}
```

## Troubleshooting tips

- Capture the full response, including headers, when escalations are needed (`request_id`, timestamp, and endpoint help support trace issues quickly).
- For login issues, confirm that the `timestamp` you signed matches `GET /api/timestamp` within ±30 seconds.
- Expect `401` responses if the bearer token is more than 30 seconds old—refresh immediately before calling protected endpoints.
- `422` validations often come from missing body parameters on `create-link`; double-check the table in the [REST endpoints](../api-reference/rest-endpoints.md#generate-payment-link) section.
- Retry idempotent operations with exponential backoff; contact [support@coinspe.com](mailto:support@coinspe.com) if server errors persist.
