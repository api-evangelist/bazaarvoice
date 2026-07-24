---
name: Fetch Bazaarvoice structured data for SEO
description: >-
  Retrieve SEO-ready structured data (JSON-LD or Microdata) for a product so review
  content is crawlable and eligible for rich results.
api: openapi/bazaarvoice-content-search-openapi.json
operations:
- getStructuredDataV2
---

# Fetch Bazaarvoice product structured data (SEO)

Use this skill to get search-engine-ready structured data for a product's UGC.

## Authentication
Send the API passkey via the `Bv-Passkey` header.

## Steps
1. **Request structured data** — call `getStructuredDataV2` with the product id as the
   `product-id` query parameter. This is the recommended structured-data endpoint.
2. **Choose the format** — the endpoint returns JSON-LD or Microdata; embed the payload
   in the product page so reviews and ratings are crawlable and eligible for rich
   results. Structured-data operations are served from seo.bazaarvoice.com.

## Conventions and error handling
- Read-only and safe to retry.
- Standard HTTP status codes; `400` on a bad/missing `product-id`, `404` when no data
  exists for the product. See errors/bazaarvoice-problem-types.yml.
