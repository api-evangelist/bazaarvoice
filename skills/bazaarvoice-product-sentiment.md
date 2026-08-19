---
name: Read AI product-sentiment features and quotes
description: Pull summarised product features, supporting quotes and expressions from the Bazaarvoice Product Sentiment API to explain what shoppers actually say about a product.
api: openapi/bazaarvoice-product-sentiment-openapi.yml
operations: [getTopFeatureQuotes, getSummarisedFeaturesWithQuotes, getAllFeaturesForProduct, getProductQuotes, getFeatureExpressions]
generated: '2026-08-13'
method: generated
source: openapi/_original/bazaarvoice-product-sentiment-openapi.json
---

# Read AI product-sentiment features and quotes

Product Sentiment turns raw review text into named product features ("battery life",
"true to size") with positive/negative weighting and the shopper quotes behind them.

## Before you start

- Base: `https://api.bazaarvoice.com/sentiment/v1/` (staging:
  `https://stg.api.bazaarvoice.com/sentiment/v1/`). Note that the version is in the path, not
  in an `ApiVersion` query parameter — this API does not follow the classic Conversations
  convention.
- Auth: passkey, requested in the Bazaarvoice Portal.

## Steps

1. **Start with the headline.** `getTopFeatureQuotes` (`GET /summarised-features`) returns the
   best and worst features for a product together with their quotes — one call is usually enough
   to render a "what shoppers say" block.
2. **Drill into one feature.** `getSummarisedFeaturesWithQuotes`
   (`GET /summarised-features/{featureId}/quotes`) returns the quotes behind a specific feature.
3. **List everything.** `getAllFeaturesForProduct` (`GET /features`) returns every feature
   detected for the product, not just the top and bottom.
4. **Raw positive quotes.** `getProductQuotes` (`GET /quotes`) returns the top positive quotes
   for a product.
5. **Underlying language.** `getFeatureExpressions` (`GET /expressions`) returns the individual
   expressions the features were derived from — use this when you need to show provenance for a
   claim rather than the summary.

## Rules that will bite you

- **This is the one Bazaarvoice API whose spec declares 429 explicitly**, with
  `application/problem+json`. Handle it: read `X-Bazaarvoice-Quota-Reset` and back off. There is
  no `Retry-After`.
- Errors are RFC 9457 (`400`, `403`, `429`) — parse `type`/`title`/`detail`.
- These are **derived** insights, not verbatim UGC. If you are attributing a claim to shoppers
  publicly, quote the returned quote rather than paraphrasing the feature label.
- Batch export of the same data is a different product: Product Sentiment Export
  (`GET /psi/v1/data`), a manifest-and-file download flow, not this API.
