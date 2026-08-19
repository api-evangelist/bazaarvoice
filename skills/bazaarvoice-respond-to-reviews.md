---
name: Respond to reviews as a brand
description: Read, create, update and delete official brand responses on Bazaarvoice reviews with the Response API.
api: openapi/bazaarvoice-response-openapi.yml
operations: [getClientResponseForReview, createClientResponseForReview, getClientResponse, updateClientResponse, deleteClientResponse, getAuthor, getReview, countResponsesForReviews]
generated: '2026-08-13'
method: generated
source:
- openapi/_original/bazaarvoice-response-openapi.json
- openapi/_original/bazaarvoice-response-count-openapi.json
---

# Respond to reviews as a brand

Use this to post and manage the brand's official reply under a review — the response that
renders as "Response from <brand>".

## Before you start

- Base host: `https://stg.api.bazaarvoice.com` for development, `https://api.bazaarvoice.com`
  for production. The Response API lives under `/response/v1/`.
- **This API does not take a passkey alone.** It uses an HTTP **bearer** token
  (`BearerAuth`). The spec's own scheme description walks a two-part login against
  `identity(-stg).portal.bazaarvoice.com/api/v1/oauth2/login` followed by a token exchange.
  A 3-legged OAuth2 flow is documented as well.
- Responses are addressed two different ways and it matters which one you hold: by
  **review** (`{client}/reviews/{reviewId}`) before a response exists, and by
  **responseGuid** afterwards.

## Steps

1. **Check what is already there.** `getClientResponseForReview`
   (`GET /response/v1/clientResponses/{client}/reviews/{reviewId}`) returns the existing
   response for that review, if any. Do this before creating — creating on top of an existing
   response is not the update path.
2. **Create.** `createClientResponseForReview`
   (`POST /response/v1/clientResponses/{client}/reviews/{reviewId}`). Keep the returned
   `responseGuid`; every later call keys off it.
3. **Edit.** `updateClientResponse` (`PATCH /response/v1/clientResponses/{responseGuid}`) —
   a PATCH, so send only the changed fields.
4. **Remove.** `deleteClientResponse` (`DELETE /response/v1/clientResponses/{responseGuid}`).
5. **Read context around a response.** `getReview`
   (`GET /response/v1/clientResponses/{responseGuid}/review`) and `getAuthor`
   (`GET /response/v1/clientResponses/{responseGuid}/author`) return the review and the
   responding author. There are matching `relationships` variants
   (`getReviewRelationship`, `getAuthorRelationship`) that return the linkage only.
6. **Bulk triage.** `countResponsesForReviews`
   (`POST /response/v1/clientResponses/reviews/-/responses`, a separate published document)
   takes a list of review ids and returns how many responses each has — use this to find the
   unanswered reviews instead of looping `getClientResponseForReview` one review at a time.

## Rules that will bite you

- **No idempotency key.** A timed-out `createClientResponseForReview` may have succeeded.
  Re-read with `getClientResponseForReview` before retrying, or you will double-post a public
  brand reply.
- The Response API returns plain HTTP status codes with a JSON body — **not**
  `application/problem+json`. Do not reuse the RFC 9457 parser you wrote for the Transactions
  or Submission APIs here.
- Rate limits are per API key and unpublished: read the `X-Bazaarvoice-QPM-*` and
  `X-Bazaarvoice-Quota-*` headers and back off from those.
