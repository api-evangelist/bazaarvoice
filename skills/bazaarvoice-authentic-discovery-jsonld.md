---
name: Make UGC visible to AI crawlers with Authentic Discovery
description: Fetch server-side JSON-LD or Microdata for a product from the Bazaarvoice Authentic Discovery API and embed it in the initial HTML so AI agents that do not execute JavaScript can read the reviews.
api: openapi/bazaarvoice-authentic-discovery-openapi.yml
operations: [getStructuredDataV2]
generated: '2026-08-13'
method: generated
source:
- openapi/_original/bazaarvoice-authentic-discovery-openapi.json
- https://developers.bazaarvoice.com/v1.0-AuthenticDiscoveryAPI/docs/home
---

# Make UGC visible to AI crawlers with Authentic Discovery

The problem this solves: Bazaarvoice's normal display integration renders reviews with
client-side JavaScript, and the AI crawlers that matter — ChatGPT, Claude, Perplexity, Gemini —
do not execute JavaScript. To them the product page has no reviews at all. Authentic Discovery
gives you the same content as structured data you can put in the **initial HTML response**.

## Before you start

- Host: `https://seo.bazaarvoice.com/structured-data/v1` in production,
  `https://seo-stg.bazaarvoice.com/structured-data/v1` for staging.
- Auth: the `Bv-passkey` **request header**. Note the lower-case `p` — the Authentic Discovery
  document spells this header differently from the Content Search document's `Bv-Passkey`.
  Send exactly what the spec for the API you are calling declares.
- This is a **server-side** call. The whole point is that the output lands in the HTML your
  origin returns, so do not call it from the browser.

## Steps

1. **Fetch.** `getStructuredDataV2` — `GET /clients/{clientId}/ugc` with the product id as a
   query parameter and your `Bv-passkey` header.
2. **Choose the format with the `Accept` header.** The endpoint returns JSON-LD, Microdata, or
   plain JSON. JSON-LD is what you want for a crawler.
3. **Embed it in the initial response.** Inline the JSON-LD in a `<script type="application/ld+json">`
   block rendered server-side. If it only appears after hydration you have not solved anything.
4. **Cache it.** UGC changes on the order of hours, not milliseconds. Cache per product and
   revalidate on your own schedule — this endpoint is rate-limited per key like everything else.
5. **Keep the JavaScript integration.** Authentic Discovery complements the bv.js display layer;
   it does not replace it. Shoppers still get the interactive components.

## Rules that will bite you

- **Do not hand-write the JSON-LD.** Emitting review markup you assembled yourself, rather than
  what Bazaarvoice returns, is how sites end up publishing rating data that does not match the
  reviews actually displayed.
- Errors are RFC 9457 `application/problem+json` on 400/401 and `default`.
- The equivalent for visual UGC is the Social Commerce Gallery Structured Data API
  (`GET /v1/structured-data/content` on `edge.curalate.com`) — a different host and a different
  API key header (`X-Curalate-Api-Key`).
