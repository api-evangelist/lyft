# Lyft GraphQL Schema

This is a conceptual GraphQL schema for the Lyft ride-sharing platform. It is derived from
the [Lyft REST API](https://developer.lyft.com/docs), public client library model types in
the [Lyft GitHub organization](https://github.com/lyft), and the publicly documented Lyft
developer surface. Lyft does not publish an official GraphQL endpoint; this schema is an
authoritative conceptual representation for tooling, federation design, and developer education.

## Schema File

- [lyft-schema.graphql](lyft-schema.graphql)

## Type Summary

The schema contains 62 named types organized across the following domains:

### Scalars (3)
`DateTime`, `Float`, `URL`

### Enums (7)
`RideTypeId`, `RideStatusCode`, `PaymentMethodType`, `VehicleColorName`, `CancelReason`, `AuthScope`, `WebhookEventType`

### Location & Geography (5)
`LatLng`, `Address`, `Location`, `Origin`, `Destination`

### Ride Product / Service Tier Types (10)
`RideType`, `Lyft`, `LyftXL`, `LyftShared`, `Standard`, `Lux`, `LuxBlack`, `LuxBlackXL`, `LyftFlexible`, `LyftPink`

### Pricing & Estimation (5)
`RideTypePricing`, `PriceQuote`, `CostEstimate`, `DriverETA`, `Surge`

### Driver & Vehicle (4)
`Driver`, `DriverRating`, `Vehicle`, `VehicleColor`

### Ride Lifecycle (7)
`Ride`, `ActiveRide`, `RideRequest`, `RideStatus`, `RideHistory`, `ScheduledRide`, `RideRoute`

### User & Passenger (4)
`UserProfile`, `Passenger`, `BusinessProfile`, `Profile`

### Business / Enterprise (2)
`BusinessRide`, `BusinessBilling`

### Payment (4)
`Payment`, `CreditCard`, `PayPalAccount`, `ApplePay`

### Ratings & Reviews (2)
`Rating`, `Review`

### Promo & Referral (4)
`Promo`, `PromoCode`, `ReferralCode`, `ReferralStatus`

### Subscription (1)
`Subscription`

### Service Zone & Heat Map (3)
`ServiceZone`, `HeatMap`, `HeatCell`

### Webhook & Events (2)
`Webhook`, `WebhookEvent`

### Auth (3)
`AccessToken`, `AuthorizationCode`, `Scope`

### Root Operations (2)
`Query`, `Mutation`

## Coverage

| Domain | Key Types |
|---|---|
| Ride types | RideType, Lyft, LyftXL, LyftShared, Standard, Lux, LuxBlack, LuxBlackXL, LyftFlexible, LyftPink |
| Pricing | RideTypePricing, CostEstimate, PriceQuote, Surge |
| ETAs | DriverETA |
| Driver | Driver, DriverRating, Vehicle, VehicleColor |
| Ride lifecycle | RideRequest, RideStatus, Ride, ActiveRide, RideHistory, ScheduledRide, RideRoute |
| Location | LatLng, Location, Address, Origin, Destination |
| Users | UserProfile, Passenger, Profile, BusinessProfile |
| Enterprise | BusinessRide, BusinessBilling |
| Payment | Payment, CreditCard, PayPalAccount, ApplePay |
| Promo | Promo, PromoCode, ReferralCode, ReferralStatus |
| Subscription | Subscription |
| Zone / demand | ServiceZone, HeatMap, HeatCell |
| Webhooks | Webhook, WebhookEvent |
| Auth | AccessToken, AuthorizationCode, Scope |

## References

- REST API Docs: https://developer.lyft.com/docs
- GitHub Organization: https://github.com/lyft
- Developer Portal: https://www.lyft.com/developers
- Concierge API: https://www.lyft.com/developers/products/concierge-api
