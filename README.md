# TerraTrack

Real-time vehicle and asset tracking on a live map. A simulated fleet drives real Riyadh roads, positions stream to the browser over SignalR, and the server watches geofences and records every trip for replay.

> **Status:** in development. See the [roadmap](#roadmap) for progress.

## Features (v1)

- **Live map:** every vehicle moves on the map in real time, with heading, speed and last-update time.
- **Fleet simulator:** a background service drives a configurable number of vehicles along real road routes, so the demo always has traffic.
- **Geofences:** draw a zone on the map; the server raises an alert the moment a vehicle enters or leaves it.
- **Trip history:** pick a vehicle and a time range to see its track and replay the trip.

## Architecture

```
 Browser (TypeScript + MapLibre GL)
     |  REST: vehicles, geofences, history
     |  SignalR: live positions, geofence alerts
     v
 ASP.NET Core 8 API
     |-- TrackingHub (SignalR)
     |-- FleetSimulator (BackgroundService)
     |-- GeofenceMonitor: enter/exit detection
     v
 PostgreSQL 16 + PostGIS
     (EF Core + Npgsql + NetTopologySuite)
```

Every simulated position goes through the same pipeline a real GPS device would: it is validated, stored in PostGIS, checked against geofences, then broadcast to connected clients. Swapping the simulator for real devices only means adding an ingestion endpoint.

## Data model

| Table | Purpose | Key columns |
|---|---|---|
| `vehicles` | Fleet registry | `id`, `name`, `type`, `color` |
| `positions` | Every GPS fix | `vehicle_id`, `recorded_at`, `geom Point(4326)`, `speed_kmh`, `heading` |
| `geofences` | Monitored zones | `id`, `name`, `geom Polygon(4326)` |
| `geofence_events` | Enter and exit alerts | `vehicle_id`, `geofence_id`, `event_type`, `occurred_at` |

Indexes: GiST on both geometry columns, and a composite index on `positions (vehicle_id, recorded_at)` for fast history queries. Trip tracks are built on the fly with `ST_MakeLine`.

## Tech stack

- **Back end:** C#, ASP.NET Core 8, SignalR, EF Core, NetTopologySuite
- **Database:** PostgreSQL 16 with PostGIS
- **Front end:** TypeScript, Vite, MapLibre GL JS, OpenStreetMap-based vector tiles
- **Tooling:** Docker Compose for local setup, GitHub Actions for CI

## Roadmap

- [ ] Project setup: solution structure, Docker Compose with PostGIS, EF Core migrations
- [ ] Fleet simulator driving vehicles along Riyadh road routes
- [ ] Live map with SignalR position streaming
- [ ] Geofences: draw, store and detect enter/exit events
- [ ] Trip history and replay
- [ ] Live demo deployment, screenshots and demo GIF

**Planned for v2:** user accounts, speed and idle alerts, fleet statistics dashboard, marker clustering for large fleets.

## Running locally

Instructions will be added with the first working version.

## License

MIT
