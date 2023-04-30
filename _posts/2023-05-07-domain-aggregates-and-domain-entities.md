---
layout: single
title: "DDD Fundamentals: Domain Aggregates and Domain Entities"
date: '2023-05-07 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
  - types
excerpt: Base classes for domain aggregates and domain entities
---

Plenty of information about [*domain aggregates*](https://www.martinfowler.com/bliki/DDD_Aggregate) and [how to design / "right size" them](https://www.youtube.com/watch?v=djq0293b2bA) already exists.  In short, a domain aggregate [represents a consistency boundary](https://www.jamesmichaelhickey.com/consistency-boundary/) for some set of data: the data within an aggregate is [kept in a valid state at all times](https://enterprisecraftsmanship.com/posts/always-valid-domain-model/).  It is *encapsulated*.  Encapsulation simplifies usage of the aggregate across the application because developers do not necessarily have to concern themselves with all the underlying business rules associated with the actions - that is, *domain commands* - that can be performed against the aggregate.

Depending on your business domain, examples of domain aggregates could include:

- a specific SKU in a product catalog...  domain commands are mostly [CRUD](https://stackify.com/what-are-crud-operations/)
- product SKU inventory in a specific warehouse...  domain commands include adding quantity and deducting quantity while correctly calculating weighted average cost
- a price sheet that lists product SKU prices for a specific client or locale...  the domain commands for the price sheet itself are mostly CRUD, but there are complex business rules determining how those prices are applied to product SKUs at time of purchase
- an order and its line items...  domain commands include modifying the order and set of line items, submitting the order for approval, approving or rejecting the order, and finalizing the order

Domain aggregates are a subset of *domain entities*: all domain aggregates are domain entities, but not all domain entities are domain aggregates.  Domain entities - and thus, domain aggregates - have three properties:

- it has an identity
- its data changes over time and persists beyond any one workflow
- its data is encapsulated

As previously mentioned, a domain aggregate represents a consistency boundary for a set of data.  Domain entities that are not aggregates belong to an aggregate; that is, there may exist one-to-one and/or one-to-many parent-child relationships between a domain aggregate and some set of domain entities.  Those domain entities are retrieved, modified, and saved alongside other data that makes up the aggregate.  It is not possible to modify a domain entity directly.

Consider the "order and its line items" example above: the order is the domain aggregate and each line item is a domain entity.  It is not possible to modify individual line items directly because doing so requires that the order's pre-tax total and total tax amounts be recalculated whenever any line item is modified.

To distinguish between domain aggregates and domain entities, I have defined two base classes: `DomainEntity<TID>` and `DomainAggregate<TID>`, where `TID` is some [identifier class](2023-04-30-domain-aggregates-and-domain-entities.md).

<script src="https://gist.github.com/RyanMarcotte/5fc7a4d91122b043eba0f960ba380461.js"></script>

<script src="https://gist.github.com/RyanMarcotte/8d99da769b2f112b78000c5fea5c7519.js"></script>

The domain aggregate and domain entity classes being declared `abstract` is an intentional implementation choice.  Many examples you will find across the Internet do not do this.  This does not mean their approach is incorrect or that my approach is correct.  As always, **it depends**.  I will dive into why `abstract` classes are used in a future post.

The above code samples include references to [*domain events*](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation), which I will not describe here.  My next post will discuss my preferred approach for both domain commands and domain events.  The code generation templates for both `DomainAggregate<TID>` and `DomainEntity<TID>` implementations will also be given at that time.
