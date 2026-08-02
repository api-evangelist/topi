---
name: Create and manage a rental offer
description: Produce a rental overview, validate and create a topi rental offer for a buyer, retrieve it, and void it if needed.
api: openapi/topi-seller-api-openapi-original.yaml
operations: [catalog#cartRentalOverview, offer#validate, offer#create, offer#show, offer#void]
---

# Create and manage a rental offer

Use this skill to turn a cart into a topi rental offer the buyer can check out.

## Auth
- OAuth2 `client_credentials`; `Authorization: Bearer $ACCESS_TOKEN`.
- Required scopes: `seller-offer:edit` (create/void), `seller-offer:read` (retrieve), `seller-catalog:read` (rental overview).

## Steps
1. **Rental overview** — call `catalog#cartRentalOverview` with the cart's products to get available tenures and monthly prices (`CartRentalOverviewResult`).
2. **Validate** — call `offer#validate` with the `CreateOfferPayload` to catch problems before creating.
3. **Create** — call `offer#create` (or `offer#createForPartner` for partner flows). The response carries `id`, `status`, and `checkout_redirect_url` — send the buyer to that URL.
4. **Track** — retrieve with `offer#show` (or `offer#showByReference` using your `seller_offer_reference`). Listen for the **Offer Updates** webhook: `accepted`, `declined`, `voided`, `pending_review`, `expired`.
5. **Void** — if the offer should be cancelled before acceptance, call `offer#void`.

## Rules
- Prefer setting a `seller_offer_reference` so you can look offers up by your own id.
- Do not blind-retry `offer#create` (no idempotency key) — on a timeout, look up by reference first.
- Errors use `application/vnd.goa.error`; 422 → `ValidationErrorList`.
