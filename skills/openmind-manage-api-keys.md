---
name: Manage OpenMind API keys and balance
description: Create, list, and delete OpenMind API keys and check the account OMCU balance via the Core API.
api: OpenMind Core API
base_url: https://api.openmind.com/api/core
operations:
  - GET /api_keys
  - POST /api_keys/create
  - POST /api_keys/delete
  - GET /account/balance
auth: Clerk-issued JWT session token as Authorization Bearer
---

# Manage OpenMind API keys and balance

Use the OpenMind **Core API** (`https://api.openmind.com/api/core`) to manage the
account's API keys and check its OMCU balance. All endpoints require a
Clerk-issued JWT session token (from the OpenMind portal or the Clerk SDK) sent
as `Authorization: Bearer <JWT>`.

## Steps

1. **Check the balance first.** `GET /account/balance` returns `plan`,
   `omcu_balance`, `monthly_unused_omcu`, `monthly_total_omcu`,
   `current_period_end`, and `cancel_at_period_end`. Confirm there are enough
   OMCU credits before issuing new keys or making metered calls.

2. **List existing keys.** `GET /api_keys` returns the active (non-deleted) keys
   with `id`, `name`, `prefix` (`om1_live_` or `om1_test_`), and `total_cost`.
   The secret portion is never returned again.

3. **Create a key when needed.** `POST /api_keys/create` returns the full
   `api_key` **once only** — store it securely. Each plan caps the number of
   keys (the Free plan allows 1; a 403 `API key limit reached` means delete one
   or upgrade the plan).

4. **Delete a key** with `POST /api_keys/delete` and body `{"id": "<key id>"}`.
   This is irreversible.

## Rules

- Handle the error envelope `{"error": "<message>"}`; map 401 → re-auth,
  403 → key-limit/plan, 404 → key not found, 429 → back off (rate limit).
- Never expose `om1_live_` keys in client-side code; prefer `om1_test_` keys and
  simulators for development.
