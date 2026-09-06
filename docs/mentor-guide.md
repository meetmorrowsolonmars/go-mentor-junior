# Food Delivery mentor guide

> This document describes how the course evolves. Domain rules, status models, and the API contract belong only in the [project specification](project.md).

## Project idea

Two students build parts of a small food-delivery system in Go. One works on the Order Service and the other on the Restaurant Service. The project is intentionally small so each lesson can focus on one microservice-engineering concern.

The first iteration produces a runnable service skeleton. Business behavior, persistence, service communication, and infrastructure are added gradually in later lessons.

Idea taken from [Design a food delivery system (DoorDash / Uber Eats)](https://www.systemdesigninterview.com/guides/system-design-interview-handbook/811-design-a-food-delivery-system-doordash-uber-eats).

## Learning objectives

After the first lesson, a student should be able to:

- create and explain a practical Go project layout;
- create one executable HTTP service;
- load basic service configuration;
- implement health checking and graceful shutdown;
- derive domain-level entities from a business and API contract;
- keep domain types separate from HTTP request and response types;
- connect HTTP routes to deterministic stub behavior;
- document how to build, run, format, and vet the service.

After the second lesson, a student should also be able to:

- implement a service's business rules with in-memory storage;
- make mutable in-memory state safe for concurrent access;
- test HTTP handlers with `httptest`;
- test application behavior using mocks for provider dependencies;
- test in-memory stores directly;
- use statement coverage as a testing guardrail;
- use the race detector to find unsafe concurrent access.

Add new objectives here only when the corresponding lesson is introduced. Planned lessons are not yet student requirements.

## Mentor-provided dependencies

Students should be able to run their assigned service without starting the rest of the system. The mentor therefore provides deterministic substitutes for dependencies that are outside the current lesson:

- **Identity Provider** recognizes seeded customers and restaurant operators.
- **Delivery Provider** creates and advances a Delivery without modeling couriers.

These substitutes may begin as in-process implementations and later be replaced by real service clients.

## Excluded from the foundation

- geographic restaurant search and delivery zones;
- customer, restaurant, and courier coordinates;
- courier registration, matching, assignment, and tracking;
- delivery-time estimation;
- payment authorization, capture, and refunds;
- promotions, coupons, taxes, tips, and variable delivery fees;
- ratings, reviews, notifications, and customer support;
- complex restaurant onboarding, approvals, and menu versioning;
- item modifiers with independent prices;
- inventory quantities and ingredient availability;
- real authentication and access tokens;
- Kafka, WebSockets, distributed transactions, and caching.

These are not hidden requirements. A later lesson may introduce one of them explicitly.

## Lessons

### Lesson 1 — Project layout

Create one runnable HTTP service for the assigned track, define the domain and transport types needed by its contract, register its routes, and return deterministic stub responses. See [Lesson 1](../lessons/01-project-layout/README.md).

### Current: Lesson 2 — In-memory business logic and tests

Implement the complete behavior of the assigned service using in-memory stores, then test its HTTP, application, and storage boundaries. See [Lesson 2](../lessons/02-in-memory-and-tests/README.md).

### Planned: Lesson 3 — Persistent storage

Add PostgreSQL repositories, migrations, transactions, and repository integration tests.

### Planned: Lesson 4 — Service communication

Connect the Order and Restaurant Services and handle timeouts, dependency errors, and contract compatibility.

### Planned: Lesson 5 — Observability

Add structured logs, metrics, and distributed traces.

The contents and order of planned lessons may change. A lesson becomes authoritative only when its own description is published in `lessons/`.

## Feature and lesson backlog

After the first five lessons, possible additions are:

1. represent service contracts with Protobuf and gRPC-Gateway and generate an OpenAPI contract;
2. introduce request idempotency and safe retries;
3. publish domain events through Kafka;
4. make event publishing reliable with a transactional outbox;
5. add Dockerfiles and Docker Compose for local startup;
6. introduce real authentication;
7. extend the Delivery lifecycle without geographic or courier-tracking logic;
8. consider payment or geographic features only after the foundation is stable;
9. collect common mistakes from student reviews and add them to the relevant support topics;
10. design self-check questions carefully and add them to support topics;
11. add focused or runnable examples only where student reviews show they are needed.
