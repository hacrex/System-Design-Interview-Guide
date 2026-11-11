# Case Study: Ride Hailing

Try it yourself first using the [blank template](../templates/design-template.md). Then compare.

## 1. Requirements

**Functional**
- A rider requests a ride (pickup + destination)
- A driver accepts or rejects a ride request
- Match the rider to the nearest available driver
- Real-time location updates (rider and driver) during the trip
- Trip tracking and status (requested, accepted, arriving, in_progress, completed)

**Non-functional**
- Low latency: match rider to driver under ~2 seconds at p99
- Real-time location updates every few seconds during a trip
- High availability: matching keeps working during a single zone failure
- Consistency: a driver cannot be assigned to two riders at once

Out of scope for this round: pricing/surge algorithms, payments, ratings/reviews, carpooling/pooling, scheduling future rides, multi-modal (bike/scooter).

## 2. Estimation

- 20 million DAU, 10% request a ride per day → 2M rides/day → ~23 rides/s average, ~100/s peak
- 500K active drivers, each sending location updates every 3 seconds → ~167K location updates/s average, higher peak
- Each ride record: ~500 bytes (rider, driver, pickup, destination, status, timestamps)
- Daily ride storage: 2M × 500 bytes = 1 GB/day — tiny
- Location updates: 167K/s × ~100 bytes = 16.7 MB/s sustained; must be handled in real time but not stored long-term

Conclusion: storage is tiny; the bottlenecks are real-time location ingestion, geo-indexing for nearest-driver lookup, and matching under surge. Keep location in memory (Redis/geo-index), persist ride state in a database.

## 3. API

```
POST   /v1/rides/request
       body: { "rider_id": "...", "pickup": { "lat": "...", "lng": "..." }, "destination": { "lat": "...", "lng": "..." }, "ride_type": "..." }
       returns: { "ride_id": "...", "status": "requested", "eta_seconds": "..." }

POST   /v1/drivers/location
       body: { "driver_id": "...", "lat": "...", "lng": "...", "available": true }
       returns: 200 OK

POST   /v1/rides/{ride_id}/accept
       body: { "driver_id": "..." }
       returns: { "ride_id": "...", "status": "accepted" }

GET    /v1/rides/{ride_id}
       returns: { "ride_id": "...", "status": "...", "driver_location": "...", "eta_seconds": "..." }
```

Location updates are high-frequency; consider websockets or a streaming endpoint for the driver app.

## 4. Data model

Access patterns:
- Update driver location frequently (write-heavy)
- Find nearest available drivers to a pickup location (geo-query, read-heavy)
- Insert/update ride state (write-heavy)
- Fetch ride status (read-heavy)

| Entity | Field | Notes |
|--------|-------|-------|
| Driver | driver_id (PK) | Unique driver identifier |
| Driver | status | online, offline, busy, available |
| DriverLocation | driver_id | Indexed |
| DriverLocation | lat, lng | Current location |
| DriverLocation | updated_at | For TTL / staleness |
| Ride | ride_id (PK) | Unique ride identifier |
| Ride | rider_id, driver_id | Indexed |
| Ride | pickup_lat, pickup_lng, destination_lat, destination_lng | Geo points |
| Ride | status | requested, accepted, arriving, in_progress, completed, cancelled |
| Ride | created_at, updated_at | Sort keys |
| Trip | ride_id | Link to ride |
| Trip | start_time, end_time, distance, fare | Trip details |

Storage choice: **Redis with geospatial indexing (GEOADD/GEORADIUS)** for live driver locations (fast, in-memory, TTL-based). **Relational database (PostgreSQL)** sharded by `rider_id` or `ride_id` for ride state — needs strong consistency so a driver isn't double-assigned.

## 5. High-level design

```
Rider App -> API Gateway -> Ride Service -> Matching Service -> Ride DB (shard by ride_id)
                                                      ^
                                              Geo Index (Redis: driver locations)
                                                      ^
Driver App -> Location Updates -> Driver Service -> Geo Index (Redis)
```

**Ride request:** rider → ride service → matching service queries geo index for nearby available drivers → picks best driver → sends ride request → driver accepts/rejects → ride service updates ride DB → rider notified.

**Location update:** driver app → driver service → update geo index (TTL ~30 seconds; stale locations expire) → matching service sees the fresh location.

**Trip tracking:** rider and driver apps subscribe to ride updates (websocket/push); ride service pushes location/status updates.

## 6. Deep dive: geo-indexing and matching

This is the hard part. How do we find the nearest available driver to a pickup in under 2 seconds?

| Approach | How it works | Pros | Cons |
|----------|--------------|------|------|
| Geohash / S2 cells | Divide map into grid cells; query nearby cells expanding outward. | Simple, fast, works with Redis sorted sets | Cells near edges may miss close drivers across cell boundaries; fixed cell size |
| Quadtree | Hierarchical spatial partitioning, adapts to density. | Handles dense and sparse areas well | More complex; rebalancing as density changes |
| PostGIS / spatial DB | SQL with geometric indexes (GiST/GiST). | Familiar, ACID | Slower for high-frequency location updates; not ideal for write-heavy live location |
| Radius search from pickup | Query drivers within increasing radii (1km, 2km, 5km...) until N found. | Simple to reason about | May query empty areas; needs careful radius tuning |

**Recommended:** Geohash or S2 cells stored in Redis sorted sets. Each driver's location is added to one or more geohash cells with `GEOADD`. To find drivers near a pickup, compute the pickup's geohash and query neighboring cells expanding outward until enough available drivers are found. Use S2 cells for better handling of cell-boundary edge cases if precision matters.

**Matching:** once candidate drivers are found, rank by (ETA to pickup, driver rating, completion rate). Send the ride request to the top driver first; if they reject/timeout, fall to the next. Use a short response deadline (e.g., 10 seconds) per driver.

**Double-booking prevention:** When a driver accepts, use a database transaction or Redis atomic operation to set the driver status to `busy`. If two requests race, only one succeeds; the other gets a retry with the next driver.

## 7. Bottlenecks and tradeoffs

- **Hot area:** Many riders in one location (concert, stadium, rain) overwhelm nearby drivers. Mitigation: surge pricing (out of scope but mention), queue riders, expand search radius, suggest nearby pickup points.
- **Location update throughput:** 167K updates/second is write-heavy. Mitigation: Redis handles it in-memory; batch updates; TTL expire stale locations; only update when location changes meaningfully.
- **Driver availability stale:** A driver goes offline but their location lingers. Mitigation: short TTL on geo entries (30s), heartbeat from driver app, mark offline on disconnect.
- **Matching race:** Two riders request simultaneously and both see the same driver. Mitigation: atomic status transition to `busy` on accept; retry with next driver for the loser.
- **Geo cell boundary:** A driver just across a cell boundary may be missed. Mitigation: query multiple neighboring cells; use S2 cells with configurable levels.
- **Real-time tracking:** During a trip, both apps need live location. Mitigation: websockets or push with location updates every 3-5 seconds; don't store every update long-term (TTL).
- **Year 3 growth:** More cities and drivers; shard geo index by city/region; add regions to Redis clusters; keep ride DB sharded.
- **Monitoring and alerts:** Track match latency (p99), match success rate, location update rate, driver acceptance rate, geo-index memory, and failed matches. Page on match latency spikes or low acceptance rate.

## Follow-up questions to practice

1. How do you handle surge pricing (dynamic pricing based on demand)?
2. How do you support carpooling (one driver, multiple riders)?
3. How do you handle a driver going offline mid-trip?
4. How do you compute ETA accurately (traffic, route)?
5. How do you scale to multiple cities with independent geo indexes?
