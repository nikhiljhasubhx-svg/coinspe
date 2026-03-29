# API Overview

Coinspe Payment Gateway exposes a concise set of endpoints for logging in, generating crypto payment links, tracking their status, and receiving webhook callbacks. This reference covers **version 1.0.0.2** of the merchant API.

## Base URLs

| Environment | Base URL |
| --- | --- |
| Production | `https://api.coinspe.com` |
| UAT | `https://uat.coinspe.com` |

Append the resource path shown in the tables below to the base URL. All bodies and responses use JSON.

## Quick reference

| # | Method | Endpoint | Description |
| --- | --- | --- | --- |
| 1 | `POST` | `/api/v1/login` | Merchant login via HMAC signature (returns JWT bearer token). |
| 2 | `GET` | `/api/timestamp` | Fetch Coinspe server time for signature drift checks. |
| 3 | `POST` | `/api/v1/payment-gateway/create-link` | Create a payment link. |
| 4 | `GET` | `/api/v1/payment-gateway/payment-link/{token}` | Fetch payment status (public). |
| 5 | `GET` | `/api/v1/payment-gateway/payment-links` | Paginated payment link history. |
| 6 | `GET` | `/api/v1/payment/crypto/market-prices` | Latest crypto market prices. |
| 7 | `GET` | `/api/v1/payment/crypto/market-prices?symbol={symbol}` | Filtered market prices for a single coin. |

## Authentication recap

- `POST /api/v1/login` requires `client_id`, `signature`, and `timestamp` headers. The signature is `HMAC-SHA256(timestamp, secret_key)`.
- Successful logins return a JWT bearer token that stays valid for **30 seconds**. Provide it as `Authorization: Bearer {token}`.
- Public endpoints (`/api/timestamp`, payment-link status lookup) do not need auth headers.

See [Authentication](../getting-started/authentication.md) for generation steps and error handling guidance.

## Request conventions

- Set `Content-Type: application/json` and `Accept: application/json` unless otherwise noted.
- Timestamps are ISO-8601 or Unix seconds depending on the field; refer to each endpoint for details.
- Pagination parameters follow `page`, `limit`, `sort_by`, and `sort_order` patterns on list endpoints.
- Validation errors return `422` along with an `errors` object; see the [Error catalog](../guides/errors.md).
