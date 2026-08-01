---
name: Search and audit Vori product changes
description: >-
  Find products fast with full-text search, review a product's change history,
  and inspect the pricing rules processed against it via the Vori REST API.
api: openapi/vori-openapi.yml
operations: [searchStoreProducts, listStoreProductHistory, listProcessedStoreProductRules, getProcessedStoreProductRule]
generated: '2026-07-21'
method: generated
---

# Search and audit Vori product changes

## Authenticate

Same bearer-token flow as every Vori API call: exchange credentials at Google
Identity Toolkit for an `idToken`, send `Authorization: Bearer <idToken>` to
`https://api.vori.com`, refresh hourly via the refresh token. Prefer a
banner-level Read-only account — every operation in this skill is a read.

## Find products

1. `searchStoreProducts` — `GET /v1/store-products/search` for fast full-text
   lookup. Note: it returns only what is indexed in Algolia and does not hit
   the database, so very recent writes may lag; fall back to
   `listStoreProducts` filters when exactness matters.

## Audit changes

1. `listStoreProductHistory` — `GET /v1/store-products/{id}/history` retrieves
   the change history for a specific store product (who/what/when for catalog
   and price changes). 404 (bodiless) means the product id is unknown.
2. Page history with the same `state` parameter contract as other list
   endpoints (`startRow`/`endRow` window, max 1000 rows, `rowCount` total).

## Inspect pricing rules

1. `listProcessedStoreProductRules` — `GET /v1/store-products/rules` lists
   product-group rules processed across store products.
2. `getProcessedStoreProductRule` — `GET /v1/store-products/{id}/rules` shows
   the processed rules for one product — use it to explain why a product's
   retail price or margin is what it is.
3. On 403 `InsufficientPermissionsError` / `NoBannerAssociationError`, the
   account is missing the role or banner association; do not retry.
