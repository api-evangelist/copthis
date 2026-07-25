---
name: Preview and place an order
description: Quote shipping for a set of line items against a Merchbar Partner API, then place the order.
api: openapi/copthis-partner-api-openapi.yml
operations: [createOrderPreview, createOrder]
---

# Preview and place an order

Always preview before placing: the preview returns availability and the shipping
options whose id the create step requires.

## Auth & conventions
- HTTP Basic over HTTPS. Success is wrapped in `data`; errors use an `error` key.
- **Not idempotent.** There is no idempotency-key mechanism. Do not blindly
  retry `createOrder` — a retry can create a second order. On a timeout, use the
  order-management flow to reconcile before retrying.

## Steps
1. **Preview** — call `createOrderPreview` (`POST /v1/order_previews`) with an
   `address` (US by default) and `line_items` (`id`, optional `variant_id`,
   `quantity`). The response echoes each line item with `available`, plus a
   `shipping_options[]` array (each with `id`, `title`, `price`, `ship_time`).
   If any item returns `available: false`, drop it or stop.
2. **Select shipping** — choose a `shipping_options[].id` from the preview.
3. **Create** — call `createOrder` (`POST /v1/orders`) with the same `address`
   and `line_items` plus `selected_shipping_option_id`. If any item is no longer
   available the whole order fails. On success `data.status` is `RECEIVED` and
   `data.id` is the order id — persist it.

## Error handling
- Non-US country on a partner without international shipping → error at preview
  or create; submit a US address.
- Item unavailable → the order fails as a unit; re-preview and retry with a valid set.
