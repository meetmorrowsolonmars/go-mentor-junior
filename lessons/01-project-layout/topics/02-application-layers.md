# Application layers

## Why they matter

The first implementation uses in-memory data and stubs. Later lessons replace them with PostgreSQL and real service clients. Clear boundaries let those mechanisms change without rewriting business behavior or HTTP handlers.

Layers are responsibilities and dependency rules, not mandatory directory names.

## A pragmatic vocabulary

- **HTTP or transport layer** registers routes, decodes requests, handles transport validation, calls business operations, and encodes responses.
- **Business-logic layer**, **application layer**, or **application service** coordinates an operation and applies business rules. An application service is code inside a deployable microservice, not another network service.
- **Domain model** represents business concepts. It may contain small, broadly useful behavior, but it does not need to contain every workflow.
- **Provider** is a broad name for an external capability. A **store** or **repository** persists data, a **client** calls another service, and a **stub** or **in-memory implementation** is a temporary substitute.

One common flow is:

```text
HTTP handler → application service → dependency interface
                                      ↑
                            concrete provider
```

HTTP details point toward application behavior. Concrete mechanisms satisfy capabilities requested by the application. The business-logic layer should not need to know whether an Order came from memory or PostgreSQL.

## Pragmatic layers and use cases

A pragmatic service often has an HTTP layer, one business-logic service layer, simple domain models, and providers. This is easy to navigate and is sufficient for many services.

A use-case-oriented design divides the application layer into operations such as `CreateOrder` or `AcceptOrder`. This can make complex workflows and their dependencies more explicit, but it also introduces more types and wiring.

Both approaches are valid. Start with the simpler design and introduce a separate use-case type only when it makes an operation easier to understand, test, or change. Clean Architecture and DDD are tools, not maturity levels that every service must reach.

## Replacing a store

Define a small interface where the application consumes it:

```go
type OrderStore interface {
	Find(ctx context.Context, id string) (Order, error)
}

type Service struct {
	orders OrderStore
}
```

Startup code can first supply an in-memory implementation:

```go
orders := memory.NewOrderStore()
app := service.New(orders)
```

A later lesson can change only that construction:

```go
orders := postgres.NewOrderStore(db)
app := service.New(orders)
```

The method bodies are not important here. The boundary matters because application code depends on the capability it needs rather than a particular storage mechanism.

## Domain and transport types

HTTP request types represent an external message. Domain types represent business meaning. They should be separate when their shapes, validation, or reasons for change differ. Reusing a simple structure can be acceptable when the meanings truly match; JSON tags alone are not a useful architecture test.

For example, a domain model might group an amount and currency because they are used together in calculations:

```go
type Money struct {
	Minor    int64
	Currency string
}

type MenuItem struct {
	Price Money
}
```

The HTTP contract can expose the same information as separate JSON fields:

```go
type menuItemResponse struct {
	PriceMinor int64  `json:"price_minor"`
	Currency   string `json:"currency"`
}

func newMenuItemResponse(item domain.MenuItem) menuItemResponse {
	return menuItemResponse{
		PriceMinor: item.Price.Minor,
		Currency:   item.Price.Currency,
	}
}
```

The domain representation can later gain validation or arithmetic without changing the API. The API representation can also evolve without forcing HTTP-specific fields into the domain. This is an illustration of separate reasons for change, not a requirement to introduce a `Money` type in Lesson 1.

Domain models may remain simple. Small methods are useful when they express reusable domain meaning:

```go
func (o Order) IsReadyForDelivery() bool {
	return o.Status == StatusReadyForPickup
}
```

Complicated workflows may remain in the application service. Do not add getters, setters, aggregates, or domain services merely to imitate a particular architecture.

## Validation boundaries

The HTTP layer checks transport concerns such as malformed JSON, missing headers, and invalid field formats. The application or business-logic layer checks business rules. Domain methods may protect small invariants that must hold wherever a type is used.

## Applying it to Lesson 1

Wire providers, application services, and handlers explicitly in the executable. A dependency-injection framework is unnecessary. Only the demonstrated request path needs to pass through all boundaries; unfinished behavior may remain deterministic stubs.

## Further reading

- Robert C. Martin's [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) explains the dependency rule. Its circles are guidance, not a required Go package tree.
- [Go Code Review Comments: Interfaces](https://go.dev/wiki/CodeReviewComments#interfaces) recommends defining small interfaces at their point of use.
- Martin Fowler's short [Domain Model](https://martinfowler.com/eaaCatalog/domainModel.html) entry provides additional background.
- Robert C. Martin's book [Clean Architecture](https://www.informit.com/store/clean-architecture-a-craftsmans-guide-to-software-structure-9780134494166) is optional extended reading.
