# Quickstart

This walkthrough gets you from raw credentials to a working payment link in four steps. Replace `{BASE_URL}` with either `https://api.coinspe.com` (production) or `https://uat.coinspe.com` (UAT).

## 1. Sync your clock

Coinspe rejects requests if the signature timestamp is more than ±30 seconds off the server clock. Call the unauthenticated timestamp endpoint and update your local time if needed:

```bash
curl '{BASE_URL}/api/timestamp'
```

Response:

```json
{
  "timestamp": 1741827600,
  "datetime": "2026-03-13 00:00:00"
}
```

## 2. Generate the HMAC signature

Create an HMAC-SHA256 signature using the Unix timestamp (seconds) as the message and your merchant secret key as the signing key:

```javascript
const timestamp = Math.floor(Date.now() / 1000).toString();
const signature = CryptoJS.HmacSHA256(timestamp, secretKey).toString();
```

Send the `client_id`, `signature`, and `timestamp` headers with every login request.

## 3. Request a bearer token

```bash
curl '{BASE_URL}/api/v1/login' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'client_id: {{client_id}}' \
  -H 'signature: {{signature}}' \
  -H 'timestamp: {{timestamp}}' \
  -d '{}'
```

Successful responses look like:

```json
{
  "message": "Login successful",
  "access_token": "eyJ0eXAi...",
  "user": { "...": "..." }
}
```

Tokens are JWTs that stay valid for 30 seconds. Refresh just before you call protected endpoints and cache the token only for the allowed window.

## 4. Create your first payment link

```bash
curl '{BASE_URL}/api/v1/payment-gateway/create-link' \
  -H 'Authorization: Bearer {{access_token}}' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "receive",
    "coin": "USDT",
    "qty": "1",
    "customer_name": "Acme Corp",
    "customer_phone": "9999999999",
    "customer_email": "billing@example.com",
    "reason": "Invoice Payment #42",
    "fees_paid_by_user": false,
    "redirect_url": "https://example.com/thank-you",
    "callback_url": "https://webhook.example.com/coinspe"
  }'
```

Check the payment status either through the public link endpoint or by registering a webhook receiver. Continue with the [Authentication guide](authentication.md) for signature rules or jump to the [REST endpoints](../api-reference/rest-endpoints.md) for full schemas.
