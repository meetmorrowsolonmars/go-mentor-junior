# Project layout

## Why it matters

A project layout helps a developer find the executable, business behavior, HTTP code, and external integrations. More importantly, package boundaries control which code can depend on which responsibilities. A large directory tree does not create a good architecture by itself.

## Go building blocks

- A **repository** is the complete version-controlled project.
- A **module** is the set of Go packages described by one `go.mod` file.
- A **package** normally contains the Go files in one directory.
- A **command** is a `package main` that builds an executable.

In this project, each service is an independent module under `services/order` or `services/restaurant`.

The `cmd/<name>` convention gives an executable a clear entry point. Code under `internal` can be imported only by packages rooted in the parent tree, so it is useful for implementation that should not become a public Go library.

## A possible starting point

A pragmatic layered layout might look like this:

```text
cmd/service/main.go
internal/api/
internal/service/
internal/domain/
internal/provider/
```

Here, `service` means the business-logic or application-service layer. Providers contain stores, external clients, and temporary in-memory implementations.

In this layout, startup code constructs the dependencies. HTTP code calls the application or business-logic layer. That layer works with domain types and small dependency interfaces. Concrete providers remain replaceable.

Start with the fewest packages that express those responsibilities. Add a directory when it has a real purpose; do not copy empty `api`, `build`, `deployments`, `pkg`, or `scripts` directories in anticipation of future lessons.

## Applying it to Lesson 1

You must be able to explain:

- where the executable starts;
- where routes and HTTP translation live;
- where application behavior belongs;
- where domain-level types live;
- where stubs and in-memory providers live;
- what depends on what.

The review evaluates those responsibilities and dependencies, not whether you copied one of the example trees.

## Further reading

- [Organizing a Go module](https://go.dev/doc/modules/layout) is the primary Go reference.
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout) is a community catalogue of common patterns. It is not an official Go standard, and its complete tree is intentionally too large for a small project.
- [Package names](https://go.dev/blog/package-names) explains how focused package names help readers understand code.
