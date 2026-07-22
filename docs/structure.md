ride-hailing-platform/
│
├── services/
│   │
│   ├── api-gateway/                     # NestJS
│   │   ├── src/
│   │   │   ├── proxy/                   # forwards /api/v1/* لكل service
│   │   │   ├── auth/                    # JWT validation middleware/guard بس (مش الـ login نفسه)
│   │   │   ├── rate-limit/
│   │   │   ├── common/filters/
│   │   │   ├── health/
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── user-service/                    # NestJS — تم init بالفعل
│   │   ├── src/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   ├── drivers/
│   │   │   ├── admin/
│   │   │   ├── common/
│   │   │   ├── config/
│   │   │   ├── health/
│   │   │   ├── kafka/
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── trip-service/                    # NestJS
│   │   ├── src/
│   │   │   ├── trips/                   # TRIP-001 → TRIP-009
│   │   │   │   ├── entities/trip.entity.ts
│   │   │   │   ├── entities/trip-event.entity.ts
│   │   │   │   └── dto/
│   │   │   ├── pricing-client/          # gRPC/REST call لـ Pricing Service
│   │   │   ├── kafka/
│   │   │   │   ├── producers/           # trip.requested, trip.started...
│   │   │   │   └── consumers/           # trip.matched, driver.rejected
│   │   │   ├── common/
│   │   │   ├── config/
│   │   │   ├── health/
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── payment-service/                 # NestJS
│   │   ├── src/
│   │   │   ├── payments/                # PAYMENT-001 → PAYMENT-005
│   │   │   │   ├── entities/transaction.entity.ts
│   │   │   │   └── dto/
│   │   │   ├── payouts/
│   │   │   │   └── entities/payout.entity.ts
│   │   │   ├── kafka/
│   │   │   │   ├── consumers/           # trip.completed (idempotent!)
│   │   │   │   └── producers/           # payment.completed/failed
│   │   │   ├── common/
│   │   │   ├── config/
│   │   │   ├── health/
│   │   │   ├── app.module.ts
│   │   │   └── main.ts
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── driver-location-service/         # Go — standard Go project layout
│   │   ├── cmd/
│   │   │   └── server/
│   │   │       └── main.go
│   │   ├── internal/
│   │   │   ├── websocket/               # WS-001, WS-004, WS-005
│   │   │   │   ├── hub.go
│   │   │   │   └── connection.go
│   │   │   ├── location/                # LOCATION-001 → LOCATION-010
│   │   │   │   ├── handler.go
│   │   │   │   └── service.go
│   │   │   ├── redisclient/
│   │   │   │   └── geo.go               # GEOADD/GEOSEARCH wrapper
│   │   │   ├── kafka/
│   │   │   │   └── producer.go          # driver.location.updated
│   │   │   ├── heartbeat/
│   │   │   │   └── monitor.go           # LOCATION-005 offline detection
│   │   │   └── config/
│   │   │       └── config.go
│   │   ├── pkg/
│   │   │   └── health/                  # /health, /ready, /live
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── matching-service/                # Go
│   │   ├── cmd/
│   │   │   └── server/
│   │   │       └── main.go
│   │   ├── internal/
│   │   │   ├── matching/                # MATCH-001 → MATCH-014
│   │   │   │   ├── engine.go            # nearest-driver logic + radius expansion
│   │   │   │   ├── lock.go              # atomic claim (MATCH-010/011)
│   │   │   │   └── retry.go             # MATCH-007, MATCH-008
│   │   │   ├── grpcclient/
│   │   │   │   └── location_client.go   # calls Driver Location Service
│   │   │   ├── kafka/
│   │   │   │   ├── consumer.go          # trip.requested
│   │   │   │   └── producer.go          # trip.matched, driver.rejected
│   │   │   └── config/
│   │   ├── pkg/health/
│   │   ├── go.mod
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── notification-service/            # Python — FastAPI
│   │   ├── app/
│   │   │   ├── main.py
│   │   │   ├── config.py
│   │   │   ├── routers/
│   │   │   │   └── health.py
│   │   │   ├── kafka/
│   │   │   │   └── consumers.py         # trip.matched, payment.completed...
│   │   │   ├── channels/                # NOTIF-005
│   │   │   │   ├── push.py
│   │   │   │   ├── sms.py
│   │   │   │   └── email.py
│   │   │   ├── models/
│   │   │   │   └── notification.py      # MongoDB document شكل
│   │   │   ├── db/
│   │   │   │   └── mongo.py
│   │   │   └── services/
│   │   │       └── dispatcher.py        # retry logic NOTIF-006
│   │   ├── requirements.txt
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── pricing-service/                 # Python — FastAPI
│   │   ├── app/
│   │   │   ├── main.py
│   │   │   ├── config.py
│   │   │   ├── routers/
│   │   │   │   ├── pricing.py           # GET /pricing/estimate
│   │   │   │   └── health.py
│   │   │   ├── services/
│   │   │   │   ├── fare_calculator.py   # PRICING-001
│   │   │   │   └── surge_calculator.py  # PRICING-002, BR-012
│   │   │   ├── kafka/
│   │   │   │   └── consumers.py         # trip.requested, driver.availability.changed
│   │   │   └── schemas/
│   │   │       └── pricing.py           # Pydantic models
│   │   ├── requirements.txt
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   └── analytics-service/               # Python — FastAPI
│       ├── app/
│       │   ├── main.py
│       │   ├── config.py
│       │   ├── routers/
│       │   │   ├── analytics.py         # trips-per-hour, eta, heatmap
│       │   │   └── health.py
│       │   ├── kafka/
│       │   │   └── consumers.py         # trip.*, driver.location.updated
│       │   ├── db/
│       │   │   └── mongo.py
│       │   └── services/
│       │       ├── eta_estimator.py     # ANALYTICS-003
│       │       └── aggregator.py        # ANALYTICS-002, ANALYTICS-004
│       ├── requirements.txt
│       ├── Dockerfile
│       └── .env.example
│
├── proto/                               # shared gRPC contracts (§13)
│   ├── location.proto                   # FindNearestDrivers
│   └── matching.proto
│
├── infra/
│   ├── docker-compose.yml               # كل الـ infra + كل الـ services
│   ├── kafka/
│   │   └── topics-init.sh               # create topics on startup
│   ├── nginx/
│   │   └── nginx.conf
│   └── k8s/                             # future (§15)
│
├── docs/
│   ├── SRS.md
│   ├── ROADMAP.md
│   └── events.md                        # نسخة تفصيلية لجدول Kafka events
│
├── .gitignore
└── README.md