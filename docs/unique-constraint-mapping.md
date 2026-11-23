# Unique-constraint mapping

Concurrent writes can raise Prisma `P2002`, but the response must identify the
resource that collided. The payment repository maps the constraint target as
follows:

| Unique target | HTTP code | API code |
| --- | --- | --- |
| `paymentRequestId` | 409 | `PAYMENT_REQUEST_CONSUMED` |
| `paymentId` | 409 | `PAYMENT_HISTORY_ALREADY_INDEXED` |
| `userId` + `stellarTxHash` | 409 | `TRANSACTION_ALREADY_INDEXED` |
| Any other payment unique target | 409 | `PAYMENT_UNIQUE_CONSTRAINT` |

The mapping is intentionally based on Prisma's constraint metadata, not on a
generic duplicate message. This keeps concurrent request races deterministic
and avoids telling clients that a payment request was consumed when the
collision was actually in transaction history. No provider error details or
secrets are included in the response.
