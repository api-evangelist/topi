---
name: Sync a product catalog to topi
description: Import a seller's products into the topi catalog so they can be priced and rented, and confirm they are supported.
api: openapi/topi-seller-api-openapi-original.yaml
operations: [catalog#checkSupported, catalog#importCatalog, catalog#createImport, catalog#getImport, catalog#listRecommendedRentalPrices]
---

# Sync a product catalog to topi

Use this skill to get a seller's products into topi so rental pricing and checkout work.

## Auth
- OAuth2 `client_credentials` at `https://identity.topi.eu/oauth2/token` (sandbox: `https://identity.topi-sandbox.eu/oauth2/token`).
- Send `Authorization: Bearer $ACCESS_TOKEN`.
- Required scopes: `seller-catalog:edit` (to import) and `seller-catalog:read` (to check/list).

## Steps
1. **Check support** — call `catalog#checkSupported` with the candidate products to confirm topi supports them for rental. (In sandbox all products are supported by default; use the `mark_product_unsupported` scenario to test the unsupported path.)
2. **Import** — for a small set call `catalog#importCatalog`. For a large catalog, create an async CSV job with `catalog#createImport` (sandbox caps at 10,000 items).
3. **Poll the job** — call `catalog#getImport` with the job id until it completes; handle the `CatalogImportJob` status.
4. **Verify pricing** — call `catalog#listRecommendedRentalPrices` to confirm products return rental prices.

## Rules
- Money is integer minor units with a currency (`MoneyAmount`: net, gross, currency).
- Errors use `application/vnd.goa.error` (`Error`: name, id, message, temporary, timeout, fault). Retry on `temporary: true` / 503; do not blind-retry writes (no idempotency-key contract).
- 422 returns a `ValidationErrorList` — fix the reported fields before retrying.
