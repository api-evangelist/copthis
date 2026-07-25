---
name: Track, update, and cancel an order
description: Check order status and tracking, update a shipping address, or cancel an order via the Merchbar Partner API.
api: openapi/copthis-partner-api-openapi.yml
operations: [getOrder, updateOrderAddress, cancelOrder]
---

# Track, update, and cancel an order

Operate on an existing order by its id (from `createOrder`).

## Auth & conventions
- HTTP Basic over HTTPS. Success wrapped in `data`; errors use an `error` key.
- Order `status` is `RECEIVED`, `SHIPPED`, or `CANCELLED`. A `tracking_number`
  is present only once `status` is `SHIPPED`.

## Steps
1. **Track** — call `getOrder` (`GET /v1/orders/{order_id}`) to read `status`,
   `tracking_number`, `ship_date`, `address`, `line_items`, and `shipping`.
2. **Update address** — call `updateOrderAddress` (`PATCH /v1/orders/{order_id}`)
   with only an `address` object (partial update; other fields are untouched).
   This fails if the change is not allowed or would change the shipping cost —
   in that case cancel and re-place the order instead.
3. **Cancel** — call `cancelOrder` (`POST /v1/orders/{order_id}/cancel`) with no
   body. The returned order has `status: CANCELLED`; some no-longer-relevant
   fields may be omitted.

## Notes
- Only `address` may be sent on the PATCH; never send other fields.
- Reconcile with `getOrder` before retrying any mutation, since the API is not
  idempotent.
