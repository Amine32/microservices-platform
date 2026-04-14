# Microservices Platform

11-service microservices platform with API gateway, JWT authentication, and domain-driven service boundaries.

![Java](https://img.shields.io/badge/Java-21-blue?style=flat&logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-brightgreen?style=flat&logo=springboot)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql)
![JWT](https://img.shields.io/badge/Auth-JWT-orange?style=flat&logo=jsonwebtokens)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker)

---

## Architecture

All client traffic enters through the API Gateway on port `8080`. The gateway routes requests by path prefix to the appropriate downstream service. Each service has its own PostgreSQL schema and validates incoming JWT tokens independently via the shared `Internship-Security` library.

```
                          ┌─────────────────────────┐
                          │       API Gateway        │
              ┌───────────│      :8080 (WebFlux)     │──────────────┐
              │           └────────────┬─────────────┘              │
              │                        │                             │
     ┌────────┴──────┐      ┌──────────┴──────────┐     ┌──────────┴────────┐
     │  User-Service │      │ Application-Service  │     │  Company-Service  │
     │    :8081      │      │       :8082           │     │      :8083        │
     └───────────────┘      └─────────────────────-┘     └───────────────────┘

     ┌───────────────┐      ┌─────────────────────-┐     ┌───────────────────┐
     │  Stack-Service│      │  Curator-Service      │     │  Season-Service   │
     │    :8084      │      │       :8085           │     │      :8086        │
     └───────────────┘      └──────────────────────┘     └───────────────────┘

     ┌───────────────┐      ┌──────────────────────┐
     │Practice-Service      │  Document-Service     │
     │    :8087      │      │       :8088           │
     └───────────────┘      └──────────────────────┘

              Shared libraries (not deployed independently):
              ┌───────────────────┐   ┌─────────────────────┐
              │ Internship-Common │   │ Internship-Security  │
              │  (DTOs, OpenAPI,  │   │  (JWT filter, IP     │
              │   exceptions)     │   │   auth, Spring Sec.) │
              └───────────────────┘   └─────────────────────┘
```

---

## Services

| Service | Port | Description |
|---|---|---|
| **Api-Gateway** | 8080 | Spring Cloud Gateway — routes all traffic by path prefix, aggregates Swagger UIs |
| **User-Service** | 8081 | Identity management — registration, login, JWT issuance, role-based access (STUDENT, SCHOOL, COMPANY, CURATOR) |
| **Application-Service** | 8082 | Candidate applications — submit/withdraw applications to positions, status tracking, interview scheduling, student group management |
| **Company-Service** | 8083 | Company and position management — CRUD for companies and open positions, paginated search |
| **Stack-Service** | 8084 | Technology catalog — languages, technologies, and stacks; CSV-seeded data |
| **Curator-Service** | 8085 | Curator management — assign curators to companies, lookup by company |
| **Season-Service** | 8086 | Recruitment calendar — season lifecycle, search periods, practice periods |
| **Practice-Service** | 8087 | Placement tracking — student internship records, contracts, semester reports, evaluations |
| **Document-Service** | 8088 | File storage — upload/download binary documents linked to entities |
| **Internship-Common** | — | Shared library: base DTOs, global exception handler, OpenAPI config |
| **Internship-Security** | — | Shared library: JWT filter, IP-based auth filter, stateless Spring Security config |

---

## Tech Stack

- **Runtime:** Java 21, Spring Boot 3.2
- **Web:** Spring MVC (REST), Spring WebFlux (gateway)
- **Security:** Spring Security, JJWT (HS256), stateless sessions
- **Data:** Spring Data JPA, PostgreSQL, Hibernate
- **Messaging:** Spring AMQP (RabbitMQ) — Application-Service
- **API Docs:** SpringDoc OpenAPI 3 — per-service Swagger UI, aggregated via gateway
- **Build:** Gradle 8 multi-project, Lombok
- **Containerization:** Docker, Docker Compose

---

## Getting Started

### Prerequisites

- Java 21+
- Docker & Docker Compose

### Run with Docker Compose

```bash
docker compose up -d
```

The API Gateway will be available at `http://localhost:8080`.

Swagger UI for each service is accessible through the gateway:

```
http://localhost:8080/user-service/swagger-ui/index.html
http://localhost:8080/company-service/swagger-ui/index.html
http://localhost:8080/application-service/swagger-ui/index.html
# ... (same pattern for other services)
```

### Build from Source

```bash
# Build all modules
./gradlew build

# Run a specific service
./gradlew :User-Service:bootRun
```

### Authentication Flow

1. `POST /user-service/api/authenticate` with `{ "email": "...", "password": "..." }` — returns a JWT
2. Include the token in all subsequent requests: `Authorization: Bearer <token>`
3. The gateway forwards the header to downstream services; each service validates it independently

---

## Project Structure

```
microservices-platform/
├── Api-Gateway/            # Spring Cloud Gateway (port 8080)
├── Application-Service/    # Application & interview domain (port 8082)
├── Company-Service/        # Company & position domain (port 8083)
├── Curator-Service/        # Curator assignment domain (port 8085)
├── Document-Service/       # File upload/download (port 8088)
├── Internship-Common/      # Shared library — DTOs, exceptions, OpenAPI
├── Internship-Security/    # Shared library — JWT, IP auth, SecurityConfig
├── Practice-Service/       # Placement & contract tracking (port 8087)
├── Season-Service/         # Recruitment calendar (port 8086)
├── Stack-Service/          # Technology catalog (port 8084)
├── User-Service/           # Auth & identity (port 8081)
├── build.gradle            # Root build config
├── settings.gradle         # Multi-module project definition
└── docker-compose.yml      # Full-stack compose configuration
```

---

## License

MIT — see [LICENSE](LICENSE).
