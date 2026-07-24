---
name: Optimize a listing's pricing with Beyond
description: Review a listing's calendar and pricing recommendations, then adjust its base price and min/max price customizations.
api: openapi/beyond-openapi-original.yml
operations: [list_listings, get_listing, list_listing_calendar, list_listing_recommendations, get_listing_base_price_customization, patch_listing_base_price_customization, get_listing_min_max_prices_customization, patch_listing_min_max_prices_customization]
---

# Optimize a listing's pricing with Beyond

Beyond is a revenue-management platform for short-term rentals. This skill reviews a
listing's demand-driven recommendations and applies pricing customizations.

## Auth
Send `Authorization: Bearer <token>` on every request. Use an OAuth2 client-credentials
token with the `listings:read` + `listings:write` scopes, or a `bpat_...` personal access
token that carries those scopes. Base URL: `https://developers.beyondpricing.com`.
Requests and responses use JSON:API (`application/vnd.api+json`).

## Steps
1. **Find the listing.** Call `list_listings` (`GET /api/v1/listings/`). Page with
   `page[number]`/`page[size]`; narrow with `filter[markets]`, `filter[enabled]`. Grab the
   listing `id`.
2. **Read the listing.** `get_listing` (`GET /api/v1/listings/{listing_id}/`) for current
   attributes; add `?include=account,owner` to pull related resources in one call.
3. **Inspect demand.** `list_listing_calendar` (`GET .../calendar/`, scope
   `reservations:read`) to see nightly prices, the per-factor `amount`, and the binding
   `effective-min-price` / `effective-max-price`.
4. **Get recommendations.** `list_listing_recommendations` (`GET .../recommendations/`) for
   Beyond's AI pricing suggestions.
5. **Read current base price.** `get_listing_base_price_customization`
   (`GET .../customizations/base-price/`).
6. **Apply changes.** `patch_listing_base_price_customization` (`PATCH .../base-price/`) and,
   if needed, `patch_listing_min_max_prices_customization` (`PATCH .../min-max-prices/`).
   These writes require `listings:write`.

## Rules
- Writes are **not** idempotent — there is no Idempotency-Key header. Do not blindly retry a
  PATCH; re-read the customization to confirm state before retrying.
- Handle `403` as a missing scope, `422` as a domain rejection (check `meta.code`), and
  `429` by honoring the `Retry-After` header. Errors arrive as a JSON:API `errors[]` array.
- Send an `X-Request-ID` header to make Beyond support debugging easier.
