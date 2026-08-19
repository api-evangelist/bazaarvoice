---
name: Send purchase transactions to trigger review requests
description: Authenticate with 2-legged OAuth2 and post single or bulk purchase transactions to Bazaarvoice so it schedules review-request emails, and invalidate a transaction when an order is cancelled.
api: openapi/bazaarvoice-transactions-openapi.yml
operations: [ingestTransaction, ingestBulkTransaction, invalidateTransaction]
generated: '2026-08-13'
method: generated
source: openapi/_original/bazaarvoice-transactions-openapi.json
---

# Send purchase transactions to trigger review requests

This is the collection side of Bazaarvoice: you tell it what a shopper bought, and it schedules
the post-interaction review-request email that produces the UGC.

## Before you start

- **This API does not accept a passkey.** It uses an HTTP **bearer** token (`accessToken`)
  obtained from **2-legged OAuth2 client credentials** — `POST /auth-v1/oauth2/token`.
  `client_id` and `client_secret` are issued by Bazaarvoice Support, not self-served from the
  portal.
- Host: `https://stg.api.bazaarvoice.com` while building. Paths sit under
  `/customer-transactions/`.
- Every transaction carries the shopper's email address. Treat this as PII from the first line
  of code: check `authentication/bazaarvoice-authentication.yml` and the Privacy API before you
  store or log a payload.

## Steps

1. **Get a token.** `POST /auth-v1/oauth2/token` with your client credentials. Cache it and
   refresh on expiry; do not fetch a token per transaction.
2. **One order at a time:** `ingestTransaction`
   (`POST /customer-transactions/transactions`). The body carries the transaction, the customer,
   and a `Product[]` array — the products determine which review requests get scheduled, so an
   order posted without its line items produces no review requests.
3. **Backfill or nightly batch:** `ingestBulkTransaction`
   (`POST /customer-transactions/bulk-transactions`). The response is a per-item result array
   (`BulkTransactionsResponseItem`) — **a 200 on the batch does not mean every item succeeded.**
   Walk the array and handle per-item failures individually.
4. **Cancellations and returns:** `invalidateTransaction`
   (`PATCH /customer-transactions/transactions/{id}`). Call this as soon as an order is
   cancelled — it is what stops a review request going out for a purchase that did not happen.

## Rules that will bite you

- **No idempotency key.** Re-posting the same order after a timeout can schedule a duplicate
  review request to a real shopper. Key your own retry ledger on your order id.
- Errors here **are** RFC 9457: `application/problem+json` on 400/401/403 and on `default`,
  with a shared `Problem` schema (`type`, `title`, `status`, `detail`, `instance`). Parse it.
- Check `TransactionStatus` and `BatchStatus` in the response rather than inferring success
  from the HTTP code.
- Rate limits are per key and unpublished; the Transactions API publishes its own rate-limit
  headers page describing the same `X-Bazaarvoice-QPM-*` family.
