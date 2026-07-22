# RideFlow — Enterprise Polyglot Ride-Hailing Backend Platform

RideFlow is a production-style ride-hailing backend built on a **polyglot microservices
architecture** — NestJS, Go, and Python — demonstrating event-driven design, real-time
geospatial tracking, and distributed systems patterns end to end.

> 📄 Full requirements & design: [`docs/SRS.md`](docs/SRS.md)
> 🗺️ Build sequence: [`docs/ROADMAP.md`](docs/ROADMAP.md)

---

## Why polyglot?

Each service is written in the language best suited to its job, not for novelty:

| Language | Used for | Why |
|---|---|---|
| **NestJS (TypeScript)** | API Gateway, User, Trip, Payment services | Structured, DTO/validation-heavy business logic; strong ecosystem for REST + ORM |
| **Go** | Driver Location, Matching services | High-concurrency, low-latency real-time workloads (goroutines over event-loop/GIL) |
| **Python (FastAPI)** | Notification, Pricing, Analytics services | Async I/O, and the natural fit for future ML (ETA prediction, surge pricing) |

---

## Architecture at a glance

```
                         ┌──────────────┐
                Clients ─▶  API Gateway │ (NestJS)
                         └──────┬───────┘
                                │
        ┌───────────┬──────────┼──────────┬───────────────┐
        ▼           ▼          ▼           ▼               ▼
  User Service  Trip Service  Payment  Driver Location  Matching Service
   (NestJS)      (NestJS)    (NestJS)   Service (Go)         (Go)
        │           │          │           │                 │
        └─────┬─────┴────┬─────┴─────┬─────┴────────┬────────┘
              ▼           ▼           ▼               ▼
          PostgreSQL     Kafka       Redis      Notification /
        (per service)  (event bus)  (geo/cache)  Pricing / Analytics
                                                    (Python/FastAPI)
                                                          │
                                                          ▼
                                                       MongoDB
```

Trip lifecycle events (`trip.requested`, `trip.matched`, `trip.completed`, ...) flow
through Kafka, decoupling services so that, for example, a Notification Service outage
never blocks payment processing. See [`docs/SRS.md §10`](docs/SRS.md) for the full event
contract table and [`docs/SRS.md §13`](docs/SRS.md) for when REST vs gRPC vs Kafka vs
WebSocket is used.

---

## Project structure

```
ride-hailing-platform/
├── services/
│   ├── api-gateway/            # NestJS
│   ├── user-service/           # NestJS
│   ├── trip-service/           # NestJS
│   ├── payment-service/        # NestJS
│   ├── driver-location-service/# Go
│   ├── matching-service/       # Go
│   ├── notification-service/   # Python / FastAPI
│   ├── pricing-service/        # Python / FastAPI
│   └── analytics-service/      # Python / FastAPI
├── proto/                      # shared gRPC contracts
├── infra/
│   ├── docker-compose.yml
│   ├── kafka/
│   ├── nginx/
│   └── k8s/                    # future
└── docs/
    ├── SRS.md                  # requirements, state machines, event/API contracts
    └── ROADMAP.md              # phase-by-phase build plan
```

---

## Tech stack

- **Frameworks:** NestJS, Go (net/http or a lightweight router), FastAPI
- **Messaging:** Apache Kafka (event bus)
- **Datastores:** PostgreSQL (transactional data), MongoDB (high-volume/unstructured
  data), Redis (geospatial index + caching)
- **Real-time:** WebSocket (driver/rider live connections)
- **Internal RPC:** gRPC + Protobuf for latency-sensitive service-to-service calls
- **Containerization:** Docker + Docker Compose (local), Kubernetes manifests (future)
- **Reverse proxy:** NGINX

---

## Getting started

> ⚠️ This project is being built incrementally, phase by phase — see
> [`docs/ROADMAP.md`](docs/ROADMAP.md) for what's currently runnable.

```bash
# 1. Clone and enter the repo
git clone <repo-url>
cd ride-hailing-platform

# 2. Copy env files for each service you're running
cp services/user-service/.env.example services/user-service/.env
# ...repeat per service as they come online

# 3. Bring up infra + services
docker compose -f infra/docker-compose.yml up -d

# 4. Check a service is healthy
curl http://localhost:3001/health
```

---

## Development workflow

Each service is developed and tested in isolation before being wired into the shared
Kafka/Redis/Postgres/Mongo infra — see the "Suggested working rhythm per phase" section
at the bottom of [`docs/ROADMAP.md`](docs/ROADMAP.md).

1. Define the API/event contract first (already captured in `docs/SRS.md`)
2. Build the service with mocked upstream dependencies
3. Wire it into `docker-compose.yml`
4. Integration-test the real flow end to end
5. Document any non-obvious decision in `docs/decisions.md`

---

## Status

🚧 Actively being built. Current phase: **Phase 1 — User Service (NestJS)**.
Track progress in [`docs/ROADMAP.md`](docs/ROADMAP.md).
