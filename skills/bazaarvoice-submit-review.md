---
name: Submit a review to Bazaarvoice
description: Fetch a submission form, authenticate the author, upload media, and submit a review through the Bazaarvoice Conversations Submission API.
api: openapi/bazaarvoice-conversations-submission-openapi.yml
operations: [GetReviewPreview, SubmitReview, AuthenticateUser, UploadPhoto, SubmitFeedback]
generated: '2026-08-13'
method: generated
source: openapi/_original/bazaarvoice-conversations-submission-openapi.json
---

# Submit a review to Bazaarvoice

Use this when a shopper writes a review on a product detail page and you are rendering
your own form rather than Bazaarvoice's hosted UI.

## Before you start

- Work against **`https://stg.api.bazaarvoice.com`** while you build. Staging auto-publishes
  submitted content roughly every 15 minutes; production moderates in 72 business hours or less.
  Switch the host to `https://api.bazaarvoice.com` only when the integration is finished.
- Authenticate with the **passkey** — `Passkey` query parameter on this API, plus
  `ApiVersion=5.4`. Keys come from API Key Management in the Bazaarvoice Portal and must be
  activated by a Technical Administrator.
- Decide the author-identity model first: **BV-mastered** (Bazaarvoice holds the account) or
  **client-mastered** (you pass a signed user id). They are not interchangeable mid-flow.

## Steps

1. **Preview the form.** Call `GetReviewPreview` (`GET /data/submitreview.json`) with
   `Action=Preview` and the `ProductId`. The response tells you which fields are required,
   which are optional, and what input type each one expects. Render the form from this
   response — do not hard-code a field list, because required fields are configured per client.
2. **Authenticate the author.** If you are client-mastered, call `AuthenticateUser`
   (`POST /data/authenticateuser.json`) to exchange your user token for the
   `UserId`/`AuthenticatedUser` value the submission needs.
3. **Upload media before the review, not after.** Call `UploadPhoto`
   (`POST /data/uploadphoto.json`) for each image. Keep the returned photo identifiers —
   the review submission references them; there is no way to attach media to an
   already-submitted review.
4. **Submit.** Call `SubmitReview` (`POST /data/submitreview.json`) with `Action=Submit`,
   the product id, the author identity, the field values, and the photo identifiers.
5. **Read the result properly.** A `200` does **not** mean the review is live. Check the
   response body: submission returns the moderation state, and in production the content is
   queued for human moderation. Surface "submitted, pending review" to the shopper, never
   "published".

## Rules that will bite you

- **There is no idempotency key.** Bazaarvoice publishes no idempotency-key mechanism anywhere
  in this estate. If a `SubmitReview` call times out, retrying can create a duplicate; duplicate
  detection is Bazaarvoice's submission workflow (device fingerprinting, moderation), not a
  contract you can rely on. Guard retries on your own side with your own submission token.
- **Errors are mixed-format.** This API declares `application/problem+json` (RFC 9457) on
  400/401/403/404, so parse `type`/`title`/`status`/`detail`. Other Bazaarvoice APIs do not —
  see `errors/bazaarvoice-problem-types.yml` before writing a shared error handler.
- **Rate limits are per key and unpublished.** Read `X-Bazaarvoice-QPM-Allotted` /
  `X-Bazaarvoice-QPM-Current` / `X-Bazaarvoice-Quota-Reset` off every response and back off from
  those; do not assume a number. There is no `Retry-After`.
- Progressive submission (`POST /data/progressiveSubmit.json`) is a different flow that lets a
  shopper submit a review in pieces. Do not mix it with the full-submission flow above.
