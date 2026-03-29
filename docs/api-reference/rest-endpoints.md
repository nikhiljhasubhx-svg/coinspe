# REST Endpoints

The sections below mirror the order of the Payment Gateway PDF so you can quickly map each requirement into your integration.

## Merchant Login

`POST {BASE_URL}/api/v1/login`

Authenticate a merchant using HMAC signature validation. Returns a bearer token that must be supplied to all protected endpoints.

### Request headers

| Header | Value | Required | Description |
| --- | --- | --- | --- |
| `Content-Type` | `application/json` | ✅ | Request body encoding. |
| `Accept` | `application/json` | ✅ | Expected response type. |
| `client_id` | `{your_client_id}` | ✅ | Merchant identifier. |
| `signature` | `{hmac_signature}` | ✅ | `HMAC-SHA256(timestamp, secret_key)`. |
| `timestamp` | `{unix_timestamp}` | ✅ | Current Unix timestamp in seconds (UTC).

### Request body

Send an empty JSON object. The backend derives the `user_id` from the signature context.

```json
{}
```

### Sample

```bash
curl '{BASE_URL}/api/v1/login' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'client_id: {{client_id}}' \
  -H 'signature: {{signature}}' \
  -H 'timestamp: {{timestamp}}' \
  -d '{}'
```

=== "Node.js"
```javascript
import crypto from 'crypto';
import axios from 'axios';

const baseUrl = process.env.COINSPE_BASE_URL;
const clientId = process.env.COINSPE_CLIENT_ID;
const secretKey = process.env.COINSPE_SECRET;
const timestamp = Math.floor(Date.now() / 1000).toString();
const signature = crypto
  .createHmac('sha256', secretKey)
  .update(timestamp)
  .digest('hex');

const response = await axios.post(
  `${baseUrl}/api/v1/login`,
  {},
  {
    headers: {
      'Content-Type': 'application/json',
      Accept: 'application/json',
      client_id: clientId,
      signature,
      timestamp,
    },
  },
);

console.log(response.data.access_token);
```

=== "Python"
```python
import os
import time
import hmac
import hashlib
import requests

base_url = os.environ["COINSPE_BASE_URL"]
client_id = os.environ["COINSPE_CLIENT_ID"]
secret_key = os.environ["COINSPE_SECRET"].encode()
timestamp = str(int(time.time()))
signature = hmac.new(secret_key, timestamp.encode(), hashlib.sha256).hexdigest()

resp = requests.post(
    f"{base_url}/api/v1/login",
    json={},
    headers={
        "Content-Type": "application/json",
        "Accept": "application/json",
        "client_id": client_id,
        "signature": signature,
        "timestamp": timestamp,
    },
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["access_token"])
```

=== "PHP"
```php
$baseUrl   = getenv('COINSPE_BASE_URL');
$clientId  = getenv('COINSPE_CLIENT_ID');
$secretKey = getenv('COINSPE_SECRET');
$timestamp = (string) time();
$signature = hash_hmac('sha256', $timestamp, $secretKey);

$ch = curl_init("{$baseUrl}/api/v1/login");
curl_setopt_array($ch, [
    CURLOPT_POST       => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'Accept: application/json',
        "client_id: {$clientId}",
        "signature: {$signature}",
        "timestamp: {$timestamp}",
    ],
    CURLOPT_POSTFIELDS => '{}',
]);
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);
echo $data['access_token'];
```

| Status | Response |
| --- | --- |
| `200 OK` | `{ "message": "Login successful", "access_token": "eyJ0eXAi...", "user": {...} }` |
| `400 Bad Request` | `{ "error": "Missing required headers" }`, `{ "error": "Invalid timestamp" }` |
| `401 Unauthorized` | `{ "error": "Invalid Client ID" }`, `{ "error": "Invalid signature" }` |

Bearer tokens expire after **30 seconds**.

## Fetch Server Time

`GET {BASE_URL}/api/timestamp`

Public endpoint that returns the server's Unix timestamp and formatted datetime. Call this before signing requests to keep drift inside the ±30 second requirement.

```bash
curl '{BASE_URL}/api/timestamp'
```

```json
{
  "timestamp": 1741827600,
  "datetime": "2026-03-13 00:00:00"
}
```

## Generate Payment Link

`POST {BASE_URL}/api/v1/payment-gateway/create-link`

Create a shareable payment link for a customer. Supports receive/send flow types, optional escrow, and custom redirect/callback URLs.

### Request headers

| Header | Value | Required |
| --- | --- | --- |
| `Authorization` | `Bearer {access_token}` | ✅ |
| `Content-Type` | `application/json` | ✅ |

### Body parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | ✅ | `"receive"` (merchant receives) or `"send"` (merchant sends). |
| `coin` | string | ✅ | Cryptocurrency symbol (e.g. `USDT`). |
| `qty` | string | ✅* | Amount in the specified coin. Required unless `amount_in_fiat` is provided. |
| `amount_in_fiat` | number | optional | Provide fiat amount instead of `qty`. |
| `customer_name` | string | ✅ | Full customer name. |
| `customer_phone` | string | ✅ | Customer phone number. |
| `customer_email` | string | optional | Customer email address. |
| `reason` | string | ✅ | Appears on the payment page. |
| `fees_paid_by_user` | boolean | ✅ | `true` = customer pays fees, `false` = merchant pays. |
| `redirect_url` | string | ✅ | URL to redirect the customer after payment. |
| `callback_url` | string | ✅ | Webhook target that receives the payment result. |
| `is_escrow` | boolean | optional | Enable escrow mode. Defaults to `false`. |
| `conversion_rate` | number | conditional | Fixed INR-per-coin rate when `is_escrow` is `true`. |

```bash
curl '{BASE_URL}/api/v1/payment-gateway/create-link' \
  -H 'Authorization: Bearer {bearer_token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "receive",
    "coin": "USDT",
    "qty": "1",
    "amount_in_fiat": "100",
    "conversion_rate": 96.39,
    "is_escrow": false,
    "customer_name": "Pradeep",
    "customer_phone": "8130919636",
    "customer_email": "billing@example.com",
    "reason": "Invoice Payment",
    "fees_paid_by_user": false,
    "redirect_url": "https://example.com",
    "callback_url": "https://webhook.example.com"
  }'
```

=== "Node.js"
```javascript
import axios from 'axios';

const payload = {
  type: 'receive',
  coin: 'USDT',
  qty: '1',
  customer_name: 'Acme Corp',
  customer_phone: '9999999999',
  reason: 'Invoice Payment',
  fees_paid_by_user: false,
  redirect_url: 'https://example.com/thank-you',
  callback_url: 'https://webhook.example.com/coinspe',
};

const res = await axios.post(
  `${process.env.COINSPE_BASE_URL}/api/v1/payment-gateway/create-link`,
  payload,
  {
    headers: {
      Authorization: `Bearer ${process.env.COINSPE_TOKEN}`,
      'Content-Type': 'application/json',
    },
  },
);

console.log(res.data.data.link_token);
```

=== "Python"
```python
import os
import requests

payload = {
    "type": "receive",
    "coin": "USDT",
    "qty": "1",
    "customer_name": "Acme Corp",
    "customer_phone": "9999999999",
    "reason": "Invoice Payment",
    "fees_paid_by_user": False,
    "redirect_url": "https://example.com/thank-you",
    "callback_url": "https://webhook.example.com/coinspe",
}

resp = requests.post(
    f"{os.environ['COINSPE_BASE_URL']}/api/v1/payment-gateway/create-link",
    json=payload,
    headers={
        "Authorization": f"Bearer {os.environ['COINSPE_TOKEN']}",
        "Content-Type": "application/json",
    },
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["data"]["link_token"])
```

=== "PHP"
```php
$payload = [
    "type" => "receive",
    "coin" => "USDT",
    "qty" => "1",
    "customer_name" => "Acme Corp",
    "customer_phone" => "9999999999",
    "reason" => "Invoice Payment",
    "fees_paid_by_user" => false,
    "redirect_url" => "https://example.com/thank-you",
    "callback_url" => "https://webhook.example.com/coinspe",
];

$ch = curl_init(getenv('COINSPE_BASE_URL') . '/api/v1/payment-gateway/create-link');
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'Authorization: Bearer ' . getenv('COINSPE_TOKEN'),
    ],
    CURLOPT_POSTFIELDS => json_encode($payload),
]);
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);
print_r($data['data']);
```

| Status | Response |
| --- | --- |
| `201 Created` | `{ "message": "Payment link created successfully.", "data": {...} }` |
| `401 Unauthorized` | `{ "message": "Unauthenticated." }` |
| `422 Unprocessable Entity` | `{ "message": "Validation failed.", "errors": {...} }` |

## Fetch Payment Status

`GET {BASE_URL}/api/v1/payment-gateway/payment-link/{link_token}`

Public endpoint (no Authorization header) that returns the current status and details of a payment link.

| Path parameter | Type | Description |
| --- | --- | --- |
| `link_token` | string | Unique payment link token (e.g. `8bdaf07a451d97f0`). |

Status values:

| Status | Meaning |
| --- | --- |
| `processing` | Link is active and awaiting payment. |
| `paid` | Payment completed successfully. |
| `failed` | Payment failed (insufficient balance or system error). |
| `expired` | Link exceeded its validity window. |

```bash
curl '{BASE_URL}/api/v1/payment-gateway/payment-link/8bdaf07a451d97f0'
```

=== "Node.js"
```javascript
import axios from 'axios';

const token = '8bdaf07a451d97f0';
const res = await axios.get(
  `${process.env.COINSPE_BASE_URL}/api/v1/payment-gateway/payment-link/${token}`,
);
console.log(res.data.data.status);
```

=== "Python"
```python
import os
import requests

token = "8bdaf07a451d97f0"
resp = requests.get(
    f"{os.environ['COINSPE_BASE_URL']}/api/v1/payment-gateway/payment-link/{token}",
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["data"]["status"])
```

=== "PHP"
```php
$token = '8bdaf07a451d97f0';
$url = getenv('COINSPE_BASE_URL') . "/api/v1/payment-gateway/payment-link/{$token}";
$response = file_get_contents($url);
$data = json_decode($response, true);
echo $data['data']['status'];
```

```json
{
  "data": {
    "id": 143,
    "link_token": "8bdaf07a451d97f0",
    "type": "receive",
    "selected_coin": "USDT",
    "status": "paid",
    "customer_name": "Pradeep",
    "customer_email": "tech.pradeep@subhx.in",
    "customer_phone": "8130919636",
    "reason": "Invoice Payment",
    "receiver_gets": "1000.00000000",
    "user_pays": "1118.00000000",
    "platform_fee": "100.00000000",
    "gst_amount": "18.00000000",
    "is_escrow": false,
    "fees_paid_by_user": true
  }
}
```

## Payment Link History

`GET {BASE_URL}/api/v1/payment-gateway/payment-links`

Returns a paginated list of payment links created by the authenticated merchant.

### Request headers

| Header | Value | Required |
| --- | --- | --- |
| `Authorization` | `Bearer {access_token}` | ✅ |
| `Content-Type` | `application/json` | ✅ |
| `Accept` | `application/json` | optional |

### Query parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `page` | integer | `1` | Page number. |
| `limit` | integer | `10` | Records per page. |
| `sort_by` | string | `created_at` | One of `created_at`, `updated_at`, `status`. |
| `sort_order` | string | `desc` | `asc` or `desc`. |

```bash
curl '{BASE_URL}/api/v1/payment-gateway/payment-links?page=1&limit=10&sort_by=created_at&sort_order=desc' \
  -H 'Authorization: Bearer {bearer_token}' \
  -H 'Accept: application/json'
```

=== "Node.js"
```javascript
import axios from 'axios';

const params = {
  page: 1,
  limit: 10,
  sort_by: 'created_at',
  sort_order: 'desc',
};

const res = await axios.get(
  `${process.env.COINSPE_BASE_URL}/api/v1/payment-gateway/payment-links`,
  {
    params,
    headers: {
      Authorization: `Bearer ${process.env.COINSPE_TOKEN}`,
      Accept: 'application/json',
    },
  },
);

console.log(res.data.meta);
```

=== "Python"
```python
import os
import requests

resp = requests.get(
    f"{os.environ['COINSPE_BASE_URL']}/api/v1/payment-gateway/payment-links",
    params={"page": 1, "limit": 10, "sort_by": "created_at", "sort_order": "desc"},
    headers={
        "Authorization": f"Bearer {os.environ['COINSPE_TOKEN']}",
        "Accept": "application/json",
    },
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["meta"])
```

=== "PHP"
```php
$query = http_build_query([
    'page' => 1,
    'limit' => 10,
    'sort_by' => 'created_at',
    'sort_order' => 'desc',
]);

$ch = curl_init(getenv('COINSPE_BASE_URL') . "/api/v1/payment-gateway/payment-links?{$query}");
curl_setopt_array($ch, [
    CURLOPT_HTTPGET => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'Authorization: Bearer ' . getenv('COINSPE_TOKEN'),
        'Accept: application/json',
    ],
]);
$response = curl_exec($ch);
curl_close($ch);

$data = json_decode($response, true);
print_r($data['meta']);
```

```json
{
  "data": [
    {
      "link_token": "8bdaf07a451d97f0",
      "type": "receive",
      "selected_coin": "USDT",
      "status": "paid",
      "customer_name": "Pradeep",
      "receiver_gets": "1000.00000000",
      "created_at": "2026-03-13T00:08:46.591480+00:00"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 10,
    "total": 42,
    "last_page": 5
  }
}
```

> Coinspe automatically expires any prior link generated for the same phone number if a new link is created before the original expires. This prevents double debits.

## Crypto Market Prices

### List all prices

`GET {BASE_URL}/api/v1/payment/crypto/market-prices`

Returns the buy and sell rates of supported cryptocurrencies relative to the default quote currency (INR).

| Header | Value | Required |
| --- | --- | --- |
| `Content-Type` | `application/json` | ✅ |
| `Accept` | `application/json` | optional |

```bash
curl '{BASE_URL}/api/v1/payment/crypto/market-prices' \
  -H 'Accept: application/json'
```

=== "Node.js"
```javascript
import axios from 'axios';

const res = await axios.get(
  `${process.env.COINSPE_BASE_URL}/api/v1/payment/crypto/market-prices`,
  { headers: { Accept: 'application/json' } },
);

console.log(res.data.prices);
```

=== "Python"
```python
import os
import requests

resp = requests.get(
    f"{os.environ['COINSPE_BASE_URL']}/api/v1/payment/crypto/market-prices",
    headers={"Accept": "application/json"},
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["prices"])
```

=== "PHP"
```php
$url = getenv('COINSPE_BASE_URL') . '/api/v1/payment/crypto/market-prices';
$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => "Accept: application/json\r\n",
    ],
]);
$response = file_get_contents($url, false, $context);
$data = json_decode($response, true);
print_r($data['prices']);
```

```json
{
  "s": "ok",
  "ts": 1774010566872,
  "quoteCurrency": "INR",
  "prices": [
    {
      "symbol": "BTCINR",
      "coin": "BTC",
      "receiveRate": 6378002.1762,
      "sendRate": 6127018.058368227
    },
    {
      "symbol": "USDT",
      "coin": "USDT",
      "receiveRate": 96.39,
      "sendRate": 91.434
    }
  ]
}
```

**Pricing logic**

- *Receive* (user sends crypto → receives fiat): rate is above market price to cover volatility and processing margin.
- *Send* (user sends fiat → receives crypto): rate is below market price to account for execution cost and liquidity spread.

```
market_price = 100
receive_rate = market_price + margin
send_rate = market_price - margin
```

### Filter by symbol

`GET {BASE_URL}/api/v1/payment/crypto/market-prices?symbol={symbol}`

| Query parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `symbol` | string | ✅ | Cryptocurrency symbol to filter (e.g. `USDT`). |

```bash
curl '{BASE_URL}/api/v1/payment/crypto/market-prices?symbol=USDT' \
  -H 'Accept: application/json'
```

=== "Node.js"
```javascript
import axios from 'axios';

const res = await axios.get(
  `${process.env.COINSPE_BASE_URL}/api/v1/payment/crypto/market-prices`,
  {
    params: { symbol: 'USDT' },
    headers: { Accept: 'application/json' },
  },
);

console.log(res.data.prices[0]);
```

=== "Python"
```python
import os
import requests

resp = requests.get(
    f"{os.environ['COINSPE_BASE_URL']}/api/v1/payment/crypto/market-prices",
    params={"symbol": "USDT"},
    headers={"Accept": "application/json"},
    timeout=10,
)
resp.raise_for_status()
print(resp.json()["prices"][0])
```

=== "PHP"
```php
$url = getenv('COINSPE_BASE_URL') . '/api/v1/payment/crypto/market-prices?symbol=USDT';
$response = file_get_contents($url);
$data = json_decode($response, true);
echo $data['prices'][0]['receiveRate'];
```

```json
{
  "s": "ok",
  "ts": 1774012214320,
  "quoteCurrency": "INR",
  "prices": [
    {
      "symbol": "USDT",
      "coin": "USDT",
      "receiveRate": 96.39,
      "sendRate": 91.434
    }
  ]
}
```
