# Patient Management System – Backend (Microservices Architecture)

This repository contains a set of Spring Boot–based microservices that collectively form the **Patient Management System**, designed with modern distributed-system patterns including:

- **Spring Boot 3.5 (Java 21)**
- **gRPC inter-service communication**
- **Kafka event streaming**
- **PostgreSQL & H2**
- **Spring Cloud Gateway (WebFlux)**
- **JWT Authentication**
- **Protocol Buffers**
- **OpenAPI / Swagger**

---

## Microservices Overview

| Service | Description | Tech |
|--------|-------------|------|
| **patient-service** | Core domain service for managing patients. Uses gRPC, Kafka, Postgres, JPA. | Spring Boot, gRPC, Kafka, JPA |
| **billing-service** | Handles billing logic; communicates via gRPC. | Spring Boot, gRPC |
| **auth-service** | Authentication, JWT issuance, user management. | Spring Security, JJWT, JPA |
| **analytics-service** | Consumes Kafka events and processes analytics. | Spring Boot, Kafka |
| **api-gateway** | Routes all external requests to internal services. | Spring Cloud Gateway (WebFlux) |


---

# Technology Stack

### Backend
- Java **17**
- Spring Boot **3.5.7**
- Spring Cloud **2025.0.0**
- Spring Web & WebFlux
- Spring Security (JWT)
- Spring Data JPA

### Communication
- **gRPC** (`grpc-netty-shaded`, `grpc-protobuf`, `grpc-stub`)
- **Protocol Buffers** (via `protobuf-maven-plugin`)
- **Kafka** (producer & consumer)

### Databases
- PostgreSQL
- H2 (testing/local)

### API Documentation
- Springdoc OpenAPI (Swagger UI)

---

# Integration Testing Module

This project includes a dedicated **integration-tests** module designed to verify the full end-to-end behavior of the system through the **API Gateway**.  
It ensures that authentication, routing, JWT validation, and service interactions behave correctly in a real running environment.


## Module Overview

**Module:** `integration-tests`  
**Purpose:** Validate real HTTP flows between client → API Gateway → backend microservices.

### Technologies Used

| Library | Purpose |
|--------|---------|
| **Rest Assured 5.3.0** | Fluent HTTP client for integration testing |
| **JUnit Jupiter 5.11.4** | Test framework |
| Java 17 | Modern language features |

"# Patient-Management-System" 
