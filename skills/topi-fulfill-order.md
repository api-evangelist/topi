---
name: Accept and fulfill a topi order
description: When a rental offer is accepted it becomes an order; accept it, acknowledge it, and create a shipment.
api: openapi/topi-seller-api-openapi-original.yaml
operations: [order#show, order#acceptOrder, order#acknowledgeOrder, shippingMethod#getShippingMethods, shipment#create]
---

# Accept and fulfill a topi order

Use this skill after a buyer accepts an offer and topi creates an order.

## Auth
- OAuth2 `client_credentials`; `Authorization: Bearer $ACCESS_TOKEN`.
- Required scopes: `seller-order:read`, `seller-order:edit`, `seller-shipping-method:read`, `seller-shipment:edit`.

## Steps
1. **Receive the order** — the **Order Events** webhook fires; fetch it with `order#show` (`FullOrderResult`).
2. **Accept** — call `order#acceptOrder` to accept the order (or `order#rejectOrder` to decline). Optionally set your own reference with `order#setMetadata`.
3. **Acknowledge** — call `order#acknowledgeOrder` to confirm you will fulfill it.
4. **Pick a shipping method** — call `shippingMethod#getShippingMethods` (create one first with `shippingMethod#create` if none exist).
5. **Ship** — call `shipment#create` with the order's assets and their serial numbers.

## Rules
- Order state transitions are enforced server-side; re-issuing `order#acceptOrder` on an already-accepted order is safe by state, not by an idempotency key.
- Include serial numbers per asset on the shipment (`Shipment` / serial-number-by-asset).
- Errors use `application/vnd.goa.error`; back off on 429/503 (`temporary: true`).
