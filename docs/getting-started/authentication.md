# Authentication

Coinspe PG couples an HMAC signature handshake with ultra-short-lived JWT bearer tokens. Implement both controls exactly as documented to avoid `401` or `Invalid Timestamp` errors.

## Auth mechanisms by endpoint

| Mechanism | Used for | Required headers |
| --- | --- | --- |
| HMAC signature | `POST /api/v1/login` | `client_id`, `signature`, `timestamp` |
| Bearer token | All protected payment-gateway endpoints | `Authorization: Bearer {token}` |
| None | Public utility endpoints (`/api/timestamp`, payment-link status) | — |

## Generating the HMAC signature

```
signature = HMAC-SHA256(timestamp, secret_key)
```

- **Message**: Unix timestamp in seconds (UTC).
- **Key**: Merchant secret key supplied by Coinspe.
- **Encoding**: Hex digest (lowercase).
- Generate a fresh signature for every login request.

Example implementations:

```javascript
const timestamp = Math.floor(Date.now() / 1000).toString();
const signature = CryptoJS.HmacSHA256(timestamp, secretKey).toString();
```

```php
$timestamp = time();
$signature = hash_hmac('sha256', $timestamp, $secretKey);
```

### Timestamp rules

- Must represent the current UTC time.
- Accepted drift between client and server clocks: **±30 seconds**.
- Use [`GET /api/timestamp`](../api-reference/rest-endpoints.md#fetch-server-time) to calibrate before generating signatures.
- Provide the raw numeric timestamp in the `timestamp` header alongside `client_id` and `signature`.

## Bearer token lifecycle

- Issued via `POST /api/v1/login`.
- JWT is valid for **30 seconds**.
- Send as `Authorization: Bearer {token}` on every protected call.
- Refresh immediately after expiry; do not reuse stale tokens.

| Status | Reason | Fix |
| --- | --- | --- |
| `400 Bad Request` | Missing headers or timestamp drift | Include all headers and sync with `/api/timestamp` before signing. |
| `401 Unauthorized` | Invalid Client ID / signature | Re-check merchant credentials and recompute the HMAC per request. |

Need to consume callbacks? Review the [Webhook guide](../api-reference/webhooks.md) for payload structure and reconciliation tips.
