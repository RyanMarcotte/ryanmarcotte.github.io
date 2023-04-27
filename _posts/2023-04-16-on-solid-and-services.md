---
layout: single
title: On SOLID and Services...
date: '2023-04-16 13:57:17 -0600'
excerpt: My thoughts on C# "service" classes and how they violate SOLID principles
---

I have worked on a few systems that implement workflows using *service interfaces* and *service classes*, forming a set of [*interface-implementation pairs*](https://martinfowler.com/bliki/InterfaceImplementationPair.html).  An example of one such interface-implementation pair is shown below.

<script src="https://gist.github.com/RyanMarcotte/fd0b49038214e37bab901f048f04740e.js"></script>

<script src="https://gist.github.com/RyanMarcotte/adab5652281d39dcc3336d7187689f88.js"></script>

The example code above is - in part - from a system I have worked on that has since been refactored to eliminate the interface-implementation pair.
{: .notice--info}

The above is an example of a *low-level service*: the operations available amount to nothing more than simple data queries and mutations.  Elsewhere in this system, there are *high-level services* that depend on the low-level services.  With this architecture, it is possible for a high-level service to call a lower-level service, which calls a still-lower-level service, and so on.

In my experience, using interface-implementation pairs in this way has many drawbacks.

**It is possible for services to depend on other services**.  The coupling between services and the reuse of service code makes some parts of the application very difficult to reason about because developers have to jump around to many different methods to determine what is going on.  Increased cognitive load reduces productivity.  Additionally, as the number of services grows, it becomes incredibly difficult to distinguish between the services that are low-level and those that are high-level: the knowledge is implicit unless you continuously look at class dependency graphs.

**Service interfaces and implementations are not [SOLID](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)**.  Every principle in SOLID is violated in some way:

- *single responsibility principle*...  services do more than one thing; in the example above, the service handles two different kinds of inventory quantity adjustments for both materials and product SKUs
- *open/closed principle*...  adding or removing service capabilities requires modifying both the service interface and its implementation
- *Liskov substitution principle*...  there is tight coupling between a service's interface and its implementation, and service behavior is complex and unique enough that a straightforward substitution (for example, like mocks for tests) is not typically possible
- *interface segregation principle*...  service interfaces define many methods; therefore, client code depends on all of those methods even though they may not be used
- *dependency inversion principle*...  components must depend on high-level abstractions and not low-level details; [interfaces are not abstractions](https://blog.ploeh.dk/2010/12/02/Interfacesarenotabstractions/)

**Services often mix business rules with infrastructure concerns**.  In the example above, the service performs data retrieval, handles data mutation according to some business rules, and commits changes made to the database.  For simple workflows, this is less of an issue, but as complexity grows [the code quickly becomes an unmaintainable mess](https://enterprisecraftsmanship.com/posts/painless-tdd/).

**Service implementations are difficult to test**.  This follows from the previous point.  Often, the services have multiple dependencies and those dependencies are mocked.  Mocks can produce divergence in behavior between production code and test code, so [they should be used sparingly](https://enterprisecraftsmanship.com/posts/when-to-mock/).  A service dependency's production code could be modified while mocks of that service are not, resulting in bugs that slip through to production.  The complexity of testing services often means that only a few unit tests are written, if any.

**Services often violate *encapsulation***.  Encapsulation is not strictly about information hiding or bundling data and operations together.  It is the act of protecting data integrity.  In the example application above, it is possible for product SKU inventory and material inventory to be linked; that is, product SKU inventory quantity changes must also affect the linked material's inventory quantity and vice versa.  The current service design requires that developers know this business rule: a developer must remember to call `SynchronizeProductSkuAndMaterialQuantity` after calling `UpdateProductSkuQuantity` or `ApplyProductSkuQuantityChange`.  Also note that there is no method for synchronizing inventory quantity changes between product SKU inventory and material inventory if `UpdateMaterialQuantity` or `ApplyMaterialChange` are called.

For the reasons stated above, I consider the use of "services" an [*anti-pattern*](https://blog.ploeh.dk/2019/01/21/some-thoughts-on-anti-patterns/).  Better implementations exist, which I will describe in future posts.
