# Health probes

## Why they matter

An orchestrator needs to know whether a process should be restarted and whether it should receive new traffic.

## Probe meanings

| Probe     | Question                                            | Typical reaction to failure                             |
| --------- | --------------------------------------------------- | ------------------------------------------------------- |
| Liveness  | Is the process alive, and could restarting it help? | Restart the container.                                  |
| Readiness | Should this instance receive new traffic?           | Remove it from service endpoints without restarting it. |
| Startup   | Has slow initialization had enough time to finish?  | Delay liveness and readiness checks.                    |

Liveness should normally describe the process itself. Making it fail because a database or another service is temporarily unavailable can cause unnecessary restart loops. Readiness may eventually depend on capabilities required to serve requests, but that distinction becomes useful only when real dependencies are introduced.

## Lesson 1 behavior

Both services expose:

- `GET /livez` for liveness;
- `GET /readyz` for readiness.

For Lesson 1, both handlers return `200 OK` with an empty body while the HTTP server is running. They perform no database, filesystem, or external-service checks. During graceful shutdown, the server closes its listeners first, so these endpoints become unavailable together with the rest of the API.

The startup probe is introduced as a concept only. Lesson 1 has no slow initialization and does not require a startup endpoint or Kubernetes configuration.

The names `/livez` and `/readyz` are course conventions, not Internet standards. Probe systems primarily act on the HTTP status code, so do not return `200 OK` with an unhealthy message in the response body.

## Applying it to Lesson 1

Register both endpoints alongside the business routes. Keep their handlers fast, deterministic, and free of sensitive diagnostic information. Kubernetes YAML and dependency-aware readiness are deferred to deployment and infrastructure lessons.

## Further reading

- [Kubernetes: Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/) provides the short conceptual overview.
- [Kubernetes: Configure Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) describes probe behavior and configuration in detail.
- [Kubernetes API health endpoints](https://kubernetes.io/docs/reference/using-api/health-checks/) shows the same distinction in Kubernetes components.
