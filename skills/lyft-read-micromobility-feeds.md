---
name: lyft-read-micromobility-feeds
description: Read live station, vehicle, pricing and alert data for Lyft's eight bikeshare and scooter systems from the public GBFS feeds, with no key and no account.
api: lyft:lyft-gbfs-feeds
operations:
  - gbfs
  - system_information
  - station_information
  - station_status
  - free_bike_status
  - system_pricing_plans
  - system_alerts
  - vehicle_types
generated: '2026-09-17'
method: generated
source: gbfs/lyft-gbfs.yml (probed 2026-09-17), https://github.com/MobilityData/gbfs/blob/master/gbfs.md
---

# Read Lyft's micromobility feeds

This is the one Lyft API surface that needs no key, no account and no business relationship. Everything is
anonymous HTTPS GET, and it speaks GBFS — so any GBFS client reads it with no bespoke connector.

## 1. Start at auto-discovery, never at a guessed path

`GET https://gbfs.lyft.com/gbfs/2.3/<system>/gbfs.json` returns the list of every feed the system serves,
per language, with absolute URLs. Follow those URLs; do not construct feed paths yourself.

Eight systems are live, all served from `gbfs.lyft.com`:

| system_id | Name | Where | GBFS |
|---|---|---|---|
| lyft_nyc | Citi Bike | New York, NY | 2.3 |
| lyft_chi | Divvy | Chicago, IL | 2.3 |
| lyft_cabi | Capital Bikeshare | Washington, DC | 2.3 |
| lyft_pdx | Biketown | Portland, OR | 2.3 |
| lyft_dca | Lyft Scooters | Washington, DC | 2.3 |
| lyft_den | Lyft Bikes & Scooters | Denver, CO | 2.3 |
| lyft_bay | Lyft Bike (Bay Wheels) | SF Bay Area, CA | **1.1** |
| lyft_bos | Bluebikes | Boston, MA | **1.1** |

Brand domains (`gbfs.citibikenyc.com`, `gbfs.divvybikes.com`, `gbfs.baywheels.com`, `gbfs.bluebikes.com`,
`gbfs.capitalbikeshare.com`, `gbfs.biketownpdx.com`) serve their own discovery document whose feed URLs all
point back at `gbfs.lyft.com`. Either entry point works.

## 2. Handle two versions of the standard

Six systems serve GBFS 2.3 and two still serve 1.1. The differences that will bite:
`vehicle_types` exists only on 2.3, and `ebikes_at_stations` appears only on the 1.1 systems. Branch on the
`version` field of the discovery document, not on the system name.

## 3. Respect ttl

Every feed carries `ttl: 60` and a `last_updated` epoch. Poll no faster than the ttl.

## 4. Check freshness before you trust a feed

`last_updated` is the freshness contract, and one system violates it: `lyft_den` advertises `ttl: 60` but
its `last_updated` was `2024-12-16T17:57:06Z` when probed on 2026-09-17 — roughly 21 months stale, while
every other system updated within a minute. Compare `last_updated` against now before showing vehicle
availability to a user, for every system, every time.

## 5. Pricing comes from the feed, not from a pricing page

`system_pricing_plans` carries the real consumer ride prices per system — unlock fee, per-minute rate,
currency — e.g. Citi Bike `EBIKE_SINGLE_RIDE` at a $4.99 unlock plus $0.41/minute, Capital Bikeshare at
$1.00 plus $0.35/minute for an e-bike. Read it from the feed; the prices differ per city and change.

## 6. Alerts

`system_alerts` carries service disruptions. Read it before reporting a station as available.
