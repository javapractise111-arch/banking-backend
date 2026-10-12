# Banking Backend

This repository contains a modular Java 21 / Spring Boot 3 banking backend structured around a ledger-first accounting model. The project is designed for enterprise-grade handling of customer onboarding, account management, transfers, statements, administration, notifications, and monitoring.

## Project Status

This implementation is intentionally structured as a production-oriented foundation with key enterprise patterns, domain rules, and infrastructure scaffolding. It covers core banking workflows and the required architectural separation, while some advanced enterprise integrations (for example, fully integrated OIDC provider setup, live production secret management, and deep operational hardening) are documented as follow-up work under the "Known gaps" section.

## Architecture

- banking-domain: domain model, value objects, enums, and rules
- banking-application: use cases, ports, DTOs, validation, and transaction orchestration
- banking-persistence: JPA entities, repository implementations, Flyway migrations, and database access
- banking-security: JWT/authn/authz, security rules, RBAC, password handling, and audit hooks
- banking-infra: outbox, notifications, metrics/tracing hooks, and operational infrastructure
- banking-api: Spring Boot application, controllers, OpenAPI, config, and entrypoint

## Core banking design

- Double-entry ledger is the authoritative financial record
- Money uses BigDecimal and a currency-aware value object
- Every financial movement is represented as a balanced journal pair
- Transfer operations cannot create or destroy value
- Idempotency keys are required for non-idempotent commands
- Reversals are explicit and preserve immutable historical records
- Insufficient funds fail in a controlled, explicit way
- Major account operations are transactional and protected by optimistic locking / row locking where appropriate

## Features included

- Customer registration and verification workflow
- Secure token-based authentication flow with RBAC-ready security model
- Account opening and lifecycle management
- Deposits, withdrawals, and transfers
- Beneficiary management
- Scheduled payment support scaffolding
- Statement and reporting primitives
- Administrative operations and audit trails
- Transactional outbox and notification hooks
- Health checks, metrics, and correlation IDs
- Flyway database migrations
- OpenAPI (Springdoc)
- Docker Compose for local Postgres

## Local development

Prerequisites:

- Java 21+
- Maven 3.9+
- Docker and Docker Compose

Build all modules:

```bash
mvn clean verify
```

Run the API locally:

```bash
mvn -pl banking-api spring-boot:run
```

Or with Docker Compose:

```bash
docker compose up -d postgres
mvn -pl banking-api spring-boot:run
```

## Environment configuration

Set environment variables, for example:

```bash
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=bankingdb
export DB_USERNAME=banking
export DB_PASSWORD=banking
export JWT_SECRET=change-me-in-production
```

Application profiles:

- default: base configuration
- local: local development values
- test: test configuration

## Database migration

The persistence module includes Flyway SQL migration scripts under `banking-persistence/src/main/resources/db/migration`.

## Example API payloads

### Customer registration

```http
POST /api/v1/customers
Content-Type: application/json

{
  "firstName": "Alice",
  "lastName": "Morgan",
  "email": "alice@example.com",
  "phone": "+1-555-0100",
  "documentNumber": "AB123456C",
  "password": "StrongPass!23"
}
```

### Deposit

```http
POST /api/v1/accounts/1001/deposits
Content-Type: application/json

{
  "amount": "2500.00",
  "currency": "USD",
  "idempotencyKey": "dep-1001-001",
  "description": "Initial credit"
}
```

### Transfer

```http
POST /api/v1/transfers
Content-Type: application/json

{
  "fromAccountId": 1001,
  "toAccountId": 1002,
  "amount": "500.00",
  "currency": "USD",
  "idempotencyKey": "xfr-2024-001",
  "description": "Client transfer"
}
```

## Known gaps and intentionally deferred work

- Full ODIC/OAuth authorization server configuration is scaffolded but not wired to a third-party identity provider in this repository
- Real KMS/secret manager integration is represented as an abstraction, not a production secret-backend implementation
- Advanced reconciliation, regulatory reporting, and mass payment workflows are not fully implemented in this initial pass
- Production hardening, independent security review, and full operational validation are required before production deployment

## Security notes

This codebase deliberately avoids unsafe patterns: service-layer financial operations are explicit, immutable journal entries are used, and account balances are never updated as a standalone destructive operation. No regulatory claim is made. Production readiness requires independent review, threat modeling, penetration testing, and environment-specific controls.

## Package structure

```text
banking-domain/
  src/main/java/com/banking/domain/

banking-application/
  src/main/java/com/banking/application/

banking-persistence/
  src/main/java/com/banking/persistence/
  src/main/resources/db/migration/

banking-security/
  src/main/java/com/banking/security/

banking-infra/
  src/main/java/com/banking/infra/

banking-api/
  src/main/java/com/banking/api/
  src/main/resources/
```

## Documentation

- `docs/adr/0001-ledger-first-accounting.md`
- `docs/threat-model.md`

## Testing

The repository includes a starting set of unit and integration tests across the domain, application, and API layers. In this environment, direct Maven execution was not possible through the API toolchain, so compile/test execution is documented as a required local validation step in a real Java build environment.
