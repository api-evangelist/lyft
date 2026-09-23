---
name: lyft-concierge-ride-for-a-passenger
description: Book and manage a Lyft ride on behalf of someone who has no Lyft account — patients, clients, employees — using the Concierge API.
api: lyft:lyft-concierge-rides-api
operations:
  - listConciergeRideTypes
  - listConciergeCostEstimates
  - createConciergeRide
  - getConciergeRide
  - getConciergeRideStatus
  - listConciergeRides
  - cancelConciergeRide
generated: '2026-09-17'
method: generated
source: openapi/lyft-concierge-rides-api-openapi.yml, openapi/lyft-ride-types-api-openapi.yml, openapi/lyft-cost-estimates-api-openapi.yml, https://help.lyft.com/business/hc/en-us/articles/360001599667-Concierge-API-overview
---

# Book a concierge ride for a passenger

The Concierge API dispatches rides for a passenger who does not have a Lyft account — the pattern behind
patient transport, client transport and employee transit. Base URL `https://api.lyft.com/v1`, bearer
token from the `client_credentials` flow against the organisation's API client.

Access is not self-serve. An organisation gets an API client through a Lyft Business relationship and
attaches it to a program in the Lyft Business Portal
(https://help.lyft.com/business/hc/en-us/articles/8587470351891-Managing-your-API-client-and-program-connections).
Cost is consumption-based — the organisation pays for rides used — and no price list is published.

## 1. Check what the program supports at that location

- `listConciergeRideTypes` (`GET /concierge/ridetypes`) with `lat`, `lng`. Concierge availability can differ
  from consumer ride types depending on how the program is configured — read it, do not assume.
- `listConciergeCostEstimates` (`GET /concierge/cost`) with `start_lat`, `start_lng`, `end_lat`, `end_lng`.
  Concierge pricing can differ from consumer pricing.

## 2. Create the ride

`createConciergeRide` (`POST /concierge/rides`) carrying the passenger details and locations. Returns
`200`; declares `400`, `401` and `422` on failure — `422` is the concierge-specific one, so surface its
description rather than collapsing it into a generic error.

**No idempotency key exists on this operation.** A retry after a timeout books a second ride for the same
passenger. If a create is ambiguous, call `listConciergeRides` (`GET /concierge/rides`, filterable by
status, time range and passenger) and reconcile before trying again.

## 3. Track

- `getConciergeRide` (`GET /concierge/rides/{id}`) — the full record.
- `getConciergeRideStatus` (`GET /concierge/rides/{id}/status`) — the lighter status-only call. Use this one
  for polling.

## 4. Reverse

`cancelConciergeRide` (`POST /concierge/rides/{id}/cancel`) is the reversal path, returning `204`.

Two boundaries the contract states plainly: a completed ride **cannot** be cancelled, and cancellation fees
may apply depending on the ride's status and how long a driver has been assigned. No numeric window is
published — do not invent one when telling a coordinator what a late cancellation will cost.

## Operating notes

- Scheduling a ride in advance widens the exposure: the earlier you book, the longer the window in which a
  passenger's plans can change, and the only remedy is `cancelConciergeRide`.
- There is no public webhook catalogue. Subscription scopes exist (`rides.subscribe_all` and friends) but no
  event schema is published, so plan for polling `getConciergeRideStatus` unless your Lyft Business contact
  has given you webhook documentation.
