# Patient Management System (Backend)

Microservices-based backend for a patient management platform, built with Spring Boot and focused on service-to-service communication, security, and event-driven workflows.

## Architecture

This repository contains the following services:

- `api-gateway` (port `4004`): Entry point for clients, routes requests and validates JWT.
- `auth-service` (port `4005`): Login endpoint, JWT generation, token validation.
- `patient-service` (port `4000`): Patient CRUD, publishes events, calls billing via gRPC.
- `billing-service` (HTTP `4001`, gRPC `9001`): Receives patient billing account creation requests via gRPC.
- `analytics-service` (default Spring Boot port unless overridden): Kafka consumer for patient events.
- `integration-tests`: End-to-end tests against the gateway.

### Runtime flow

1. Client logs in through gateway: `POST /auth/login`.
2. Gateway forwards to `auth-service`.
3. Client calls protected patient API: `GET|POST|PUT|DELETE /api/patients...`.
4. Gateway runs `JwtValidation` filter (`auth-service /validate`).
5. `patient-service` persists patient data, calls `billing-service` via gRPC, and sends Kafka event.
6. `analytics-service` consumes the Kafka patient event.

## Tech stack

- Java `17`
- Spring Boot `3.5.7`
- Spring Cloud Gateway `2025.0.0`
- Spring Security + JWT (`jjwt`)
- Spring Data JPA
- gRPC + Protocol Buffers
- Kafka
- H2 (local/dev defaults) and PostgreSQL dependency for external DB usage
- Maven

## Repository structure

```text
.
|-- api-gateway/
|-- auth-service/
|-- patient-service/
|-- billing-service/
|-- analytics-service/
|-- integration-tests/
|-- api-requests/         # HTTP request samples
`-- grpc-requests/        # gRPC request sample files
```

## Prerequisites

- Java `17+` installed and on `PATH`
- Maven `3.9+` (or use each module's `mvnw` / `mvnw.cmd`)
- Kafka running for event flow (`patient-service` producer + `analytics-service` consumer)

Optional:
- PostgreSQL if you want external DB instead of embedded defaults

## Required environment variables

Some properties are expected at runtime and are not fully defined in committed `application.properties`.

Set these before running:

```powershell
$env:AUTH_SERVICE_URL="http://localhost:4005"
$env:JWT_SECRET="REPLACE_WITH_BASE64_ENCODED_SECRET"
$env:SPRING_KAFKA_BOOTSTRAP_SERVERS="localhost:9092"
```

Notes:
- `api-gateway` expects `auth.service.url` (map from `AUTH_SERVICE_URL`).
- `auth-service` expects `jwt.secret` (map from `JWT_SECRET`). It must be Base64-encoded and long enough for HMAC signing.
- Kafka bootstrap servers should be set for `patient-service` and `analytics-service`.
- `patient-service` gRPC billing target defaults to `localhost:9001` via:
  - `billing.service.address` (default `localhost`)
  - `billing.service.grpc.port` (default `9001`)

## Running locally

Start services in this order to avoid startup/runtime dependency errors.

1. `auth-service`
2. `billing-service`
3. `patient-service`
4. `analytics-service`
5. `api-gateway`

Example (PowerShell, from repo root):

```powershell
cd .\auth-service; .\mvnw.cmd spring-boot:run
cd ..\billing-service; .\mvnw.cmd spring-boot:run
cd ..\patient-service; .\mvnw.cmd spring-boot:run
cd ..\analytics-service; .\mvnw.cmd spring-boot:run
cd ..\api-gateway; .\mvnw.cmd spring-boot:run
```

## API usage

All client-facing calls should go through gateway at `http://localhost:4004`.

### 1) Login

`POST /auth/login`

```json
{
  "email": "testuser@test.com",
  "password": "password123"
}
```

The seeded auth user is created from `auth-service/src/main/resources/data.sql`.

### 2) Get patients (protected)

`GET /api/patients`

Header:

```http
Authorization: Bearer <token>
```

### 3) Create patient (protected)

`POST /api/patients`

```json
{
  "name": "John Doe",
  "email": "john.doe@demo.com",
  "address": "123 Main St",
  "dateOfBirth": "1995-09-09",
  "registeredDate": "2026-02-15"
}
```

## API docs

- Patient OpenAPI through gateway: `http://localhost:4004/api-docs/patients`
- Auth OpenAPI through gateway: `http://localhost:4004/api-docs/auth`

## Testing

### Unit tests

Run per service:

```powershell
cd .\patient-service; .\mvnw.cmd test
```

### Integration tests

`integration-tests` assumes the system is already running (especially gateway on `4004`).

```powershell
cd .\integration-tests
mvn test
```

## Biome (repo formatting/linting)

This repository includes a root `biome.json` for consistent lint/format on supported file types (for example JSON/JSONC/JavaScript if added).

Run without installing globally:

```powershell
npx @biomejs/biome check .
npx @biomejs/biome format --write .
```

## Troubleshooting

- `401 Unauthorized` from patient routes:
  - Ensure `Authorization: Bearer <token>` is sent.
  - Ensure `AUTH_SERVICE_URL` points to running `auth-service`.
- Gateway startup failure for missing property:
  - Set `AUTH_SERVICE_URL`.
- Auth startup failure for JWT:
  - Set valid Base64 `JWT_SECRET`.
- Kafka publish/consume issues:
  - Verify `SPRING_KAFKA_BOOTSTRAP_SERVERS` and broker availability.
- gRPC connection refused in `patient-service`:
  - Ensure `billing-service` is running and listening on port `9001`.

## License

Add your project license here (for example MIT/Apache-2.0).
