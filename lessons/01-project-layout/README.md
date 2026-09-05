# Lesson 1: project layout

## Goal

Create exactly one runnable HTTP server for the assigned service and provide a stub implementation of its [API contract](../../docs/project.md).

This lesson covers project organization, domain types, route wiring, and a simple request path. Business rules, validation, persistence, and real service communication are deferred.

Each student completes the common tasks and one assigned service track.

## Support material

These optional pages provide starting points for unfamiliar topics:

- [Project layout](topics/01-project-layout.md)
- [Application layers](topics/02-application-layers.md)
- [HTTP server lifecycle](topics/03-server-lifecycle.md)
- [Health probes](topics/04-health-probes.md)
- [Development commands](topics/05-development-commands.md)

## Common tasks

1. Initialize a Go module for the assigned service.
2. Choose and create the project layout.
3. Create one executable that starts one HTTP server.
4. Read the HTTP listen address from `HTTP_ADDR`, with a documented default.
5. Shut the server down gracefully when the process receives an interrupt signal.
6. Implement `GET /livez` and `GET /readyz`; both must return `200 OK` with an empty body while the service is running.
7. Derive the service's domain types from the project and API contract.
8. Register every API route owned by the assigned service.
9. Return responses using the field names and shapes from the contract.
10. Use deterministic in-process stubs for unfinished application behavior, persistence, and external dependencies.
11. Implement at least one complete path from an HTTP request to a deterministic in-process result.
12. Keep domain types and application behavior independent from HTTP request and response types.
13. Add documented commands to build, run, format, and vet the service.
14. Document local startup and configuration in the service README.

The internal package structure, interface design, constructors, libraries, and dependency-wiring approach are the student's decisions.

## Track A: Order Service

1. Register all Cart, Customer Order, and Restaurant Order routes.
2. Define the domain types needed by those routes and status models.
3. Supply deterministic in-process Restaurant and Delivery substitutes so the service starts without external servers.
4. Demonstrate at least one Customer request that returns a valid contract response.
5. Demonstrate at least one Restaurant request that returns a valid contract response.
6. Return stub responses for the remaining routes.

## Track B: Restaurant Service

1. Register all Restaurant and Menu routes.
2. Define the domain types needed by those routes.
3. Supply deterministic in-process Restaurant and Menu data so the service starts without persistence.
4. Demonstrate at least one read request that returns a valid contract response.
5. Demonstrate at least one `PUT` request that returns a valid contract response.
6. Return stub responses for the remaining routes.

## Acceptance criteria

1. The assigned service builds successfully.
2. The documented formatting and vetting commands succeed.
3. Exactly one service process and one HTTP server are created.
4. The service starts without a database or another running service.
5. `GET /livez` and `GET /readyz` return `200 OK` with empty bodies.
6. Every assigned route is registered and accepts its documented HTTP method.
7. Stub responses use the contract's JSON field names and value types.
8. At least one assigned endpoint executes a complete request-to-stub-to-response path.
9. The process shuts down gracefully.
10. The repository contains no committed credentials or machine-specific absolute paths.

## Not required

- business-rule checks or domain validation;
- complete Cart, Order, Restaurant, Menu, or Delivery behavior;
- unit, integration, or end-to-end tests;
- a database schema, migrations, or persistent storage;
- real communication between the two student services;
- real authentication;
- retries, telemetry, Kafka, or containers.

The two implementations will be completed and connected in later lessons using the same shared API contract.
