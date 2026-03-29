# Webhooks

When you create a payment link you must provide a `callback_url`. Coinspe will POST the final payment outcome to that URL whenever the link settles (paid or failed).

## Delivery format

```
POST {callback_url}
Content-Type: application/json
```

### Sample payload

```json
{
  "event": "payment.completed",
  "link_token": "8bdaf07a451d97f0",
  "status": "paid",
  "coin": "USDT",
  "receiver_gets": "1000.00000000",
  "user_pays": "1118.00000000",
  "customer_name": "Pradeep",
  "customer_email": "tech.pradeep@subhx.in",
  "timestamp": "2026-03-13T00:52:17.000000+00:00",
  "escrow_fee": "0.04000000",
  "platform_fee": "2.10000000",
  "applied_fee": "2.10000000",
  "gst_amount": "0.38000000",
  "tds_amount": "0.00000000",
  "type": "receive"
}
```

(Fields such as `debit_transaction_id` / `credit_transaction_id` may be present when applicable.)

## Processing recommendations

- Treat the callback as a notification only. Always re-fetch the payment link via [`GET /api/v1/payment-gateway/payment-link/{token}`](rest-endpoints.md#fetch-payment-status) and reconcile with your own records before fulfilling orders.
- Log the entire payload along with request headers for auditing and replay protection.
- Implement exponential backoff when retrying your own downstream workflows; Coinspe retries failed deliveries up to 10 times.
- Coinspe automatically expires earlier active links for the same phone number if a new link is created, preventing duplicate debits.

## Sample receiver

=== "Node.js (Express)"
```javascript
import crypto from 'crypto';
import express from 'express';

const app = express();
app.use(express.json({ type: '*/*' }));

app.post('/webhooks/coinspe', (req, res) => {
  const secret = process.env.COINSPE_WEBHOOK_SECRET;
  const body = JSON.stringify(req.body);
  const signature = crypto
    .createHmac('sha256', secret)
    .update(body)
    .digest('hex');

  if (signature !== req.header('X-Coinspe-Signature')) {
    return res.status(400).send('Invalid signature');
  }

  // Re-fetch payment status before acting
  console.log('Payment event:', req.body);
  res.sendStatus(200);
});

app.listen(3000, () => console.log('Listening for Coinspe webhooks'));
```

=== "Python (FastAPI)"
```python
import hmac
import hashlib
import os
from fastapi import FastAPI, Header, HTTPException, Request

app = FastAPI()

@app.post("/webhooks/coinspe")
async def coinspe_webhook(request: Request, x_coinspe_signature: str = Header(None)):
    body = await request.body()
    secret = os.environ["COINSPE_WEBHOOK_SECRET"].encode()
    expected = hmac.new(secret, body, hashlib.sha256).hexdigest()

    if expected != x_coinspe_signature:
        raise HTTPException(status_code=400, detail="Invalid signature")

    payload = await request.json()
    # TODO: re-fetch payment status via the public endpoint
    print("Coinspe event", payload)
    return {"received": True}
```
