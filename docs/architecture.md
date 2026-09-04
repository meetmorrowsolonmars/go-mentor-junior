# Food Delivery architecture views

> Mentor working document. These diagrams explain service ownership and interactions; they do not prescribe Go packages, domain structs, or database tables.

C4 describes people, software systems, containers, and components. It does not describe domain-entity cardinality or behavior over time, so the C4 views below are followed by a domain relationship view and sequence diagrams.

## C4: system context

```mermaid
flowchart LR
    customer["Person: Customer"]
    operator["Person: Restaurant operator"]
    system["Software system: Food Delivery"]

    customer -->|"Browses menus, manages cart, places and reads orders"| system
    operator -->|"Maintains restaurant and menu, processes orders"| system
```

## C4: container view

```mermaid
flowchart LR
    customer["Person: Customer"]
    operator["Person: Restaurant operator"]

    subgraph foodDelivery["Software system: Food Delivery"]
        orderService["Container: Order Service — carts, orders, state transitions, delivery orchestration"]
        restaurantService["Container: Restaurant Service — restaurants, menus, prices, availability"]
        deliveryService["Container: Delivery Service stub — delivery lifecycle without couriers or location"]
    end

    customer -->|"Browse restaurants and menus"| restaurantService
    customer -->|"Manage cart and orders"| orderService
    operator -->|"Maintain restaurant and menu"| restaurantService
    operator -->|"Accept, reject, and prepare orders"| orderService
    orderService -->|"Read authoritative restaurant and menu data"| restaurantService
    orderService -->|"Create delivery and start delivery"| deliveryService
```

## C4: logical component view

The components are responsibilities, not required packages or interfaces.

```mermaid
flowchart TB
    subgraph orderService["Container: Order Service"]
        customerApi["Component: Customer API — cart, create order, history, cancellation"]
        restaurantOrderApi["Component: Restaurant order API — accept, reject, prepare, ready"]
        cartRules["Component: Cart behavior — single restaurant, quantities, limits"]
        orderWorkflow["Component: Order workflow — checkout, ownership, state transitions"]
        restaurantAccess["Component: Restaurant access — current menu and restaurant data"]
        deliveryAccess["Component: Delivery access — create and advance delivery"]

        customerApi --> cartRules
        customerApi --> orderWorkflow
        restaurantOrderApi --> orderWorkflow
        cartRules --> restaurantAccess
        orderWorkflow --> restaurantAccess
        orderWorkflow --> deliveryAccess
    end

    subgraph restaurantService["Container: Restaurant Service"]
        publicApi["Component: Public restaurant API — restaurant and menu reads"]
        operatorApi["Component: Restaurant operator API — restaurant and menu writes"]
        restaurantRules["Component: Restaurant and menu behavior — ownership, currency, availability"]

        publicApi --> restaurantRules
        operatorApi --> restaurantRules
    end

    restaurantAccess -->|"HTTP contract"| publicApi
    deliveryAccess -->|"Delivery contract"| deliveryStub["Delivery Service stub"]
```

## Domain relationships

This view shows conceptual relationships only. Students choose their own domain and persistence models.

```mermaid
erDiagram
    CUSTOMER ||--o| CART : owns
    CUSTOMER ||--o{ ORDER : places
    RESTAURANT ||--o{ MENU_ITEM : offers
    RESTAURANT ||--o{ ORDER : receives
    CART ||--o{ CART_LINE : contains
    CART_LINE }o--|| MENU_ITEM : selects
    ORDER ||--|{ ORDER_LINE : contains
    ORDER_LINE }o--|| MENU_ITEM : references
    ORDER ||--o| DELIVERY : requires_after_acceptance
```

Important ownership rules:

- Restaurant Service owns Restaurant and Menu Item.
- Order Service owns Cart, Cart Line, Order, and Order Line.
- Delivery Service owns Delivery.
- Order Line preserves historical item data even though it references a Menu Item identifier.
- One accepted Order has exactly one Delivery, addressed by `order_id`.

## Sequence: add an item and create an order

```mermaid
sequenceDiagram
    actor Customer
    participant OrderService as Order Service
    participant RestaurantService as Restaurant Service

    Customer->>OrderService: Set cart item
    OrderService->>RestaurantService: Get restaurant and menu item
    RestaurantService-->>OrderService: Current availability and price
    OrderService-->>Customer: Updated cart

    Customer->>OrderService: Create order with delivery address
    OrderService->>RestaurantService: Revalidate restaurant and menu
    RestaurantService-->>OrderService: Current authoritative data
    OrderService->>OrderService: Create PENDING order and clear cart
    OrderService-->>Customer: Created order
```

If validation or creation fails, no Order is created and the Cart remains unchanged.

## Sequence: accept and prepare an order

```mermaid
sequenceDiagram
    actor Operator as Restaurant operator
    participant OrderService as Order Service
    participant DeliveryService as Delivery Service

    Operator->>OrderService: Accept PENDING order
    OrderService->>DeliveryService: Create delivery using order_id
    DeliveryService-->>OrderService: WAITING_FOR_PREPARATION
    OrderService->>OrderService: Change order to ACCEPTED
    OrderService-->>Operator: Accepted order

    Operator->>OrderService: Start preparation
    OrderService->>OrderService: Change order to PREPARING
    OrderService-->>Operator: Preparing order

    Operator->>OrderService: Mark order ready
    OrderService->>DeliveryService: Start delivery using order_id
    DeliveryService-->>OrderService: IN_DELIVERY
    OrderService->>OrderService: Change order to READY_FOR_PICKUP
    OrderService-->>Operator: Ready order
```

If Delivery creation fails, the Order remains `PENDING`. If starting Delivery fails, the Order remains `PREPARING`.
