# Specification changelog

This file records student-visible changes to the project specification, API contract, and active lessons.

## Unreleased

- No changes.

## 0.1.0 - 2026-09-05

- Defined the Order Service and Restaurant Service boundaries.
- Defined Cart, Order, Restaurant, Menu Item, and Delivery behavior.
- Defined the initial HTTP operations, error codes, and status models.
- Added separate Order Service and Restaurant Service tracks for Lesson 1.
- Added optional support topics for project layout, application layers, HTTP server lifecycle, health probes, and development commands.
- Replaced the ambiguous `GET /healthz` Lesson 1 requirement with separate `GET /livez` and `GET /readyz` endpoints.
- Standardized the configurable HTTP listen address as `HTTP_ADDR` for both services.
