---
name: lyft-request-a-ride
description: Request, track, amend and cancel a Lyft ride for an authenticated rider, using the published Rides API operations.
api: lyft:lyft-rides-api
operations:
  - listRideTypes
  - listETAs
  - listCostEstimates
  - createRide
  - getRide
  - updateRideDestination
  - cancelRide
  - getRideReceipt
generated: '2026-09-17'
method: generated
source: openapi/lyft-rides-api-openapi.yml, openapi/lyft-ride-types-api-openapi.yml, openapi/lyft-eta-api-openapi.yml, openapi/lyft-cost-estimates-api-openapi.yml, conventions/lyft-conventions.yml
---

# Request a Lyft ride

Base URL `https://api.lyft.com/v1`. Every call carries `Authorization: Bearer <token>`.
Ride creation and anything that reads a specific rider needs a three-legged (`authorization_code`) token;
`listRideTypes`, `listETAs`, `listCostEstimates` and `listNearbyDrivers` accept a two-legged
(`client_credentials`) token. The scope vocabulary is in `scopes/lyft-scopes.yml`; `rides.request`,
`rides.read` and `rides.active_ride` are the ones this flow needs.

## 1. Quote before you dispatch

1. `listRideTypes` (`GET /ridetypes`) with `lat` and `lng` — what is actually available at the origin.
2. `listETAs` (`GET /eta`) with `lat`, `lng`, optional `ride_type` and destination — seconds to pickup.
3. `listCostEstimates` (`GET /cost`) with `start_lat`, `start_lng`, `end_lat`, `end_lng` — price range.

These three are read-only and safe to retry. They are also the only rehearsal available: there is no
dry-run or test mode on `createRide`.

## 2. Dispatch — the one step that is not retryable

`createRide` (`POST /rides`) with a ride type and an origin; a destination is optional at request time.

**There is no idempotency key on this operation.** The contract defines no `Idempotency-Key` header and
Lyft publishes no replay protection. A retried `createRide` after a timeout will dispatch a second driver
and charge a second fare. On a timeout or an ambiguous failure, do not retry: call `listRides`
(`GET /rides`, filtered by `start_time`) and look for the ride you may already have created.

Handle `400` (invalid parameters) and `401` (bad or expired token) — those are the only failure responses
the contract declares. No `429` and no `5xx` is declared, and no `Retry-After` header is documented, so
choose your own backoff.

## 3. Track

`getRide` (`GET /rides/{id}`) for the full ride record. Poll on a cadence you pick — the contract publishes
no rate limit and no rate-limit headers, so stay conservative.

## 4. Amend

`updateRideDestination` (`PUT /rides/{id}/destination`) changes the drop-off while the ride is live. It is
self-reversing: set it again. There is no undo.

## 5. Reverse

`cancelRide` (`POST /rides/{id}/cancel`) is the reversal path for `createRide`. Returns `204`.

The spec states: if the ride has already been matched with a driver, a cancellation fee may apply
depending on how long the driver has been en route. **Lyft publishes no numeric free-cancellation
window.** Do not assume one, and tell the user a fee may apply rather than quoting a threshold.

## 6. Settle

`getRideReceipt` (`GET /rides/{id}/receipt`) once the ride is complete.

## What this API will not tell you

- No error schema — a `4xx` carries a prose description and nothing else.
- No request-id or correlation header to quote in a support ticket.
- No pagination metadata on `listRides`: `limit` and `offset` only, with no total or `has_more`.
