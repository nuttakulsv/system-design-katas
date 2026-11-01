# Prompt 05: Ride dispatch

## Prompt

"Design the part of a ride-hailing app that matches riders with nearby drivers."

## Hidden requirements

- Operates in Bangkok and five other Thai cities. Traffic matters more than straight-line distance.
- Drivers send location every 4 seconds while online.
- A rider should see a matched driver within 10 seconds, or a clear "no drivers" answer.
- Drivers must not be offered two rides at once. A driver can decline; then the next driver gets the offer.
- Surge pricing exists but is a separate team; you need to expose supply and demand per area.

## Numbers

- 200k online drivers at peak; that's 50k location updates per second.
- 20k ride requests per minute at peak (rush hour, rain).
- Location data older than 30 seconds is useless.

## A strong design covers

- **Location ingestion** to an in-memory geo index (geohash or H3 cells, or Redis GEO), partitioned by city or cell, with TTL so stale drivers disappear. Not the main database.
- **Matching:** find candidates in nearby cells, rank by ETA (from a routing service or precomputed travel-time estimates, not straight-line distance), offer to one driver at a time with a short timeout, and lock the driver while offered (atomic state change: available → offered).
- **State machine** for driver and trip, persisted durably; location stays ephemeral.
- **Batching** in heavy demand: match in short windows (for example every 2 seconds) to optimise globally instead of greedy one-by-one.
- **Supply and demand counters** per cell for the surge team, via a stream.
- **Failure handling:** a city partition down shouldn't take others; drivers' apps buffer updates.

## Probes

- "Two riders request at the same moment and the same driver is closest to both."
- "The driver's phone goes into a tunnel for 40 seconds."
- "How do you know your ETA estimates are any good?"

## Curveballs

- "It starts raining across Bangkok: demand triples in 10 minutes, supply drops."
- "We're adding shared rides: two riders, one car."

## Level notes

- **Senior:** a sensible geo index, an atomic offer and lock, the trip state machine.
- **Staff:** separates the ephemeral location plane from the durable trip plane, partitions by geography, discusses batching versus greedy matching and how to measure matching quality.
- **Principal:** connects matching to marketplace health (driver earnings, rider wait times, cancellation rates), experimentation strategy, and city-by-city rollout.

## Pitfalls

- Writing every location update to a relational database.
- Straight-line distance in Bangkok traffic.
- No lock, so a driver gets two offers.
