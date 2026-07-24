---
name: Onboard a user and connect a channel account
description: Create a Beyond user, connect a channel/PMS account, refresh it to sync listings, and confirm the sync.
api: openapi/beyond-openapi-original.yml
operations: [create_user, get_user, create_account, refresh_account, get_account, list_accounts, list_listings]
---

# Onboard a user and connect a channel account

For partners managing multiple Beyond users: provision a user, attach a channel account
(Airbnb, PMS, ...), and trigger the sync that imports their listings.

## Auth
Requires an **app-level** OAuth2 client-credentials token with `user:write` and
`accounts:read`. Base URL `https://developers.beyondpricing.com`; JSON:API media type.

## Steps
1. **Create the user.** `create_user` (`POST /api/v1/users/`, scope `user:write`). A duplicate
   email returns `409 Conflict` — on 409, look the user up with `list_users`
   (`filter[email]=...`) instead of retrying the create.
2. **Confirm.** `get_user` (`GET /api/v1/users/{user_id}/`).
3. **Connect an account.** `create_account`
   (`POST /api/v1/users/{user_id}/accounts/`) with the channel credentials.
4. **Sync.** `refresh_account`
   (`POST /api/v1/users/{user_id}/accounts/{account_id}/refresh/`). If a sync is already in
   progress this returns `409 Conflict` — wait, do not hammer it.
5. **Poll status.** `get_account` (`GET .../accounts/{account_id}/`) until its sync status
   completes, or subscribe to the `account.refreshed` webhook instead of polling.
6. **Verify listings.** `list_listings` (`GET /api/v1/listings/`) to confirm the account's
   listings imported.

## Rules
- Prefer the `account.created` / `account.refreshed` webhooks (Standard Webhooks, HMAC-SHA256,
  verify the `webhook-signature` over raw bytes, de-dupe on `webhook-id`) over polling.
- `refresh_account` is a long-running background sync; treat `409` as "already running", not
  an error to retry immediately.
