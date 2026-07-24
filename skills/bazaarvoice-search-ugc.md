---
name: Search Bazaarvoice UGC (reviews and questions)
description: >-
  Search and retrieve user-generated content - reviews, questions, answers, and
  contributor profiles - from the Bazaarvoice Content Search API for a product.
api: openapi/bazaarvoice-content-search-openapi.json
operations:
- searchReviewsUsingPOST
- getReviewById
- searchQuestionsUsingPOST
- getQuestionsUsingPOSTAndQuestionId
- searchContributors
- catalogLookaheadSearch
---

# Search Bazaarvoice user-generated content

Use this skill to pull ratings, reviews, and Q&A for a product from the Bazaarvoice
Content Search API.

## Authentication
Send the API passkey on every request as the `Bv-Passkey` header. Passkeys are issued
in the Bazaarvoice Portal (API Key Management) and must be activated by a Technical
Administrator. Optionally send `X-Correlation-ID` to trace a request.

## Steps
1. **Find the product** — call `catalogLookaheadSearch` with the shopper's query to
   resolve a product id from the client catalog.
2. **Search reviews** — call `searchReviewsUsingPOST` with the product id and paging
   parameters (`Offset`, `Limit`); read `TotalResults` and iterate to page through the
   full set. Use `getReviewById` to fetch a single review by its legacy internal id.
3. **Search questions and answers** — call `searchQuestionsUsingPOST` for the product,
   then `getQuestionsUsingPOSTAndQuestionId` to expand answers on a specific question.
4. **Look up contributors** — call `searchContributors` (or `contributorLookaheadSearch`)
   to resolve author profiles referenced by the returned UGC.

## Conventions and error handling
- Page with `Offset`/`Limit`; the response carries `TotalResults`.
- Respect rate limits: read `X-Bazaarvoice-QPM-Allotted` / `X-Bazaarvoice-Quota-Allotted`
  and back off when `X-Bazaarvoice-QPM-Current` approaches the allotment. On a 429/limit
  rejection the reason is in `X-Bazaarvoice-Error-Detail`.
- Standard HTTP status codes: `400` bad request, `401` unauthorized (check the passkey),
  `404` not found, `500` server error. See errors/bazaarvoice-problem-types.yml.
- There is no client idempotency key; search operations are read-only and safe to retry.
