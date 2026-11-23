# Payment retry contract

Clients may retry a request after a timeout, lost response, or transient
network failure. Retries must reuse the same identifier and must not create a
new payment intent.

## Create and authorize

- Send the same `paymentRequestId` or external `requestId` on a retry of
  `POST /v1/payments`.
- A request ID is unique. If another request consumed it first, the API returns
  `409 PAYMENT_REQUEST_CONSUMED`; do not generate a replacement payment for the
  same NFC request.
- Authorization is a one-way `CREATED -> AUTHORIZED` transition. A repeated
  authorization attempt receives `409 PAYMENT_INVALID_STATE` and must not
  trigger another WebAuthn ceremony for a different payment.

## Submit and reconcile

- Retry `POST /v1/payments/:id/submit` with the same payment ID and signed
  envelope only when the client did not receive the previous response.
- A `CONFIRMED` payment is idempotent and returns its existing result.
- A `SUBMITTED` payment is reconciled using its stored Stellar transaction hash;
  the server does not relay a second transaction.
- A `FAILED` or expired payment is terminal. Create a new payment only from a
  fresh, unconsumed request and show the user the new intent details.

## What clients should log

Record the payment ID, request ID, API status/code, and retry attempt locally.
Never log private keys, signed XDR, access tokens, or raw provider responses.
Horizon remains authoritative for a submitted transaction, and a database
retry does not undo an on-chain transfer. These examples describe mocked and
testnet behavior; they are not proof of a live settlement.
