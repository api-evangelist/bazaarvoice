---
name: Manage email notification opt-in and opt-out lists
description: Read and write shopper email subscription state for Bazaarvoice notification emails using the Notifications Subscriptions API.
api: openapi/bazaarvoice-notifications-subscriptions-openapi.yml
operations: [fetchOptInByPage, fetchOptOutByPage, subscribe, unsubscribe]
generated: '2026-08-13'
method: generated
source:
- openapi/_original/bazaarvoice-notifications-subscriptions-openapi.json
- postman/bazaarvoice-notifications-subscriptions-api.postman_collection.json
---

# Manage email notification opt-in and opt-out lists

Bazaarvoice sends notification emails to shoppers — review requests, "your review was
published", answers to your question. This API lets a client manage those subscriptions on the
shopper's behalf instead of relying only on the unsubscribe link in the email.

## Before you start

- Host: `https://api.bazaarvoice.com` (staging `https://stg.api.bazaarvoice.com`).
  Paths sit under `/notifications/{client}/subscriptions/`.
- Auth: the `passkey` **query parameter** (the spec's `apiKey` scheme). Bazaarvoice's docs state
  the Enterprise package is a prerequisite for access to this API.
- A **shared secret key** is used separately to encrypt/decrypt email addresses. You are handling
  shopper email addresses in every call on this API — treat the whole surface as PII.

## Steps

1. **Read the opt-in list.** `fetchOptInByPage`
   (`GET /notifications/{client}/subscriptions/by_page/{emailType}/OPT_IN`) with `limit` and
   `offset`. Page with offset/limit; there is no cursor.
2. **Read the opt-out list.** `fetchOptOutByPage`
   (`GET /notifications/{client}/subscriptions/by_page/{emailType}/OPT_OUT`), same shape.
3. **Subscribe.** `subscribe` (`POST /notifications/{client}/subscriptions/subscribe`) with the
   `userToken` and `emailType`. Returns the updated or inserted subscription.
4. **Unsubscribe.** `unsubscribe` (`POST /notifications/{client}/subscriptions/unsubscribe`),
   same parameters.

## Rules that will bite you

- **`emailType` is not global.** Opt-out is per email type, so unsubscribing a shopper from one
  type does not stop the others. If a shopper asks to stop all Bazaarvoice email, iterate the
  types.
- **Consent is the point of this API — do not automate a subscribe.** `subscribe` writes an
  opt-in on a real person's behalf. Only call it from an action the shopper actually took, and
  keep your own record of when and where.
- No idempotency key, and these are POSTs. Re-read the list before retrying a failed write.
- Errors are plain HTTP status codes with a JSON body — not `application/problem+json`.
- Bazaarvoice publishes a first-party Postman collection for this API; it is checked into
  `postman/` in this repo with the staging host already set.
