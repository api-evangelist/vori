---
name: Manage Vori store products
description: >-
  Create, read, update, and restore products in a Vori grocery store catalog
  via the Vori REST API, honoring its bearer-token auth and state-parameter
  pagination contract.
api: openapi/vori-openapi.yml
operations: [listStoreProducts, getStoreProduct, createStoreProduct, updateStoreProduct, restoreStoreProduct]
generated: '2026-07-21'
method: generated
---

# Manage Vori store products

## Authenticate

1. Exchange the operator's Vori email/password at Google Identity Toolkit
   (`identitytoolkit/v3/relyingparty/verifyPassword`) for an `idToken`; send it
   as `Authorization: Bearer <idToken>` on every request to
   `https://api.vori.com`.
2. Tokens expire after one hour — refresh with the returned `refreshToken` at
   `securetoken.googleapis.com/v1/token` instead of re-sending credentials, and
   store the new refresh token each time (it may rotate).
3. For read-only integrations, use a banner-level **Read-only** role account
   (per the provider's own guidance) so accidental writes fail fast.

## Read the catalog

1. `listStoreProducts` — `GET /v1/store-products` with a URL-encoded `state`
   JSON object. Only `startRow`, `endRow`, `filterModel`, and `sortModel` are
   accepted; request at most 1000 rows per call and page with
   `startRow`/`endRow`, using the response `rowCount` as the total.
2. Scope to a store with `filterModel.store_id` (set filter) and to live items
   with `filterModel.active` (boolean filter). Default sort is `created_at`
   descending.
3. `getStoreProduct` — `GET /v1/store-products/{id}` for the full product,
   including tax rates, modifiers, barcodes, vendor products, and inventory.

## Write safely

1. `createStoreProduct` — `POST /v1/store-products`. Expect typed 400s for
   duplicate/invalid barcodes, departments, tax rates, modifiers, units of
   measure, and variable weights (see `errors/vori-problem-types.yml`).
2. A `SoftDeletedBarcodeConflictError` means a deleted product owns the
   barcode — call `restoreStoreProduct` (`POST /v1/store-products/{id}/restore`)
   instead of creating a duplicate.
3. `updateStoreProduct` — `PATCH /v1/store-products/{id}`; 404 (bodiless) means
   the id does not exist.
4. There is **no idempotency-key contract**: never blind-retry a `POST` after a
   timeout — re-list/search first to confirm whether the product was created.
5. 403 `InsufficientPermissionsError` / `NoBannerAssociationError` mean the
   account lacks the role or banner association — surface to a human rather
   than retrying.
