---
name: Sync a partner's stores and merchandise
description: Poll a Merchbar Partner API implementation for stores and their merchandise to keep catalog, availability, and pricing current.
api: openapi/copthis-partner-api-openapi.yml
operations: [listStores, getStore, listStoreMerch]
---

# Sync a partner's stores and merchandise

Use this to mirror a partner's catalog into Merchbar. Merchbar polls each store
and its merch at least once a day to keep status and quantities current.

## Auth & conventions
- HTTP Basic over HTTPS (`Authorization: Basic <base64(token:)>`). Reject non-TLS.
- Success responses are wrapped in a `data` key; collections add a `pagination`
  object (`current_page`, `per_page`, `total_pages`). A page beyond `total_pages`
  returns 200 with an empty `data` array.
- Errors return a 4xx/5xx with an `error` key (see errors/copthis-problem-types.yml).

## Steps
1. **List stores** — call `listStores` (`GET /v1/stores`), paging with `page` and
   `per_page` until `data` is empty. Each store has `id`, `title`, `visible`.
   Skip / hide stores where `visible` is false.
2. **Get store detail** — for each store id, call `getStore`
   (`GET /v1/stores/{id}`) for `description` and `photo`.
3. **List merchandise** — call `listStoreMerch` (`GET /v1/stores/{id}/merch`),
   paging to completion. For each item read `price`/`currency` (price is a string
   with no currency symbol), `quantity`, `status` (`available` | `backorder` |
   `preorder`, with `available_date` for the latter two), `photos`, and any
   `variants` (each variant carries its own `quantity`/`price`). Respect
   `visible: false` and `sale_price`.

## Notes
- Prices are strings ("15.99"); do not parse as floats where precision matters.
- Dates are ISO 8601 UTC.
