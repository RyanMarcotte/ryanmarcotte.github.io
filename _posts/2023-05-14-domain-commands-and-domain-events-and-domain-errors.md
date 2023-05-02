---
layout: single
title: "Implementing Domain-Driven Design: Domain Commands, Domain Events, and Domain Errors"
date: '2023-05-14 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
  - types
excerpt: How to apply changes to domain aggregates and respond to those changes
---

In my previous post, we established that *domain aggregates* represent a consistency boundary.  Data in the aggregate is encapsulated; in other words, it is kept in a valid state at all times.  My preferred method of preserving encapsulation is to not use `public` setters.

<script src="https://gist.github.com/RyanMarcotte/860889afb5ca2f4d899f795e76c2eab5.js"></script>

With the setters now `protected`, it is now impossible for workflows to put the aggregate's data in an invalid state.

If you are not implementing aggregates as `abstract` classes, then the setters would be declared `private` instead.
{: .notice--info}

With no `public` setters, workflows must use *domain commands* to modify the aggregate.  A *domain command* is a request to perform some action against an aggregate.  The request may succeed or fail depending on the business rules or technical constraints that we need to enforce.  If the request succeeds, one or more [*domain events*](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation) are produced.  If the request fails, one or more *domain errors* are produced.  I will describe domain commands, domain events, and domain errors individually in turn, but first I must introduce a functional programming concept.

A full explanation of functional programming and its merits are beyond the scope of this post, but - suffice to say - I was introduced to functional programming and it has completely transformed the way I write C# code.  Gone are the days where I use `null` for "no value" and use exceptions for controlling program flow.  For domain modelling, I now exclusively use `Option<T>` and `Result<TSuccess, TFailure>`: `Option<T>` for values that may or may not exist, and `Result<TSuccess, TFailure>` for operations that can succeed or fail.  [I reserve the use of exceptions for exceptional - that is, unrecoverable - error scenarios](https://enterprisecraftsmanship.com/posts/exceptions-for-flow-control/).  The [`Functional` library](https://github.com/JohannesMoersch/Functional) provides zero-allocation implementations of `Option<T>` and `Result<TSuccess, TFailure>`, but other libraries and implementations do exist.

I am an active contributor to the `Functional` library.
{: .notice--info}

Domain commands are implemented as `public` methods on the domain aggregate.  The method accepts a [*parameter object*](https://refactoring.guru/introduce-parameter-object) and returns `Result<Unit, DomainError<TID>>`.  The general implementation of a domain command is as follows:

- validate the input and the current state of the aggregate
- if successful, apply state changes and cache one or more domain events inside the aggregate
- if the input is invalid or the input would cause an invalid state change, return a domain error

<script src="https://gist.github.com/RyanMarcotte/0695a7d09aa7992f352839ce5590b970.js"></script>

`Unit` is how functional programming represents "no value".  The keyword `void` does not work.  We still need to return something.
{: .notice--info}

The `Functional` library automatically handles implicit conversion from `Unit` or `DomainError<TID>` to `Result<Unit, DomainError<TID>>`.
{: .notice--info}

Domain command parameter objects are implemented as immutable classes.  Simple!

<script src="https://gist.github.com/RyanMarcotte/b745cba5812c0b44c8c8cf01f3c90f72.js"></script>

Note that domain command parameter objects have no base class.  There is no requirement to have one because there is no shared schema between individual domain command parameter objects.

Domain events are also implemented as immutable classes.  They inherit from a `DomainEvent<TID>` base class.

<script src="https://gist.github.com/RyanMarcotte/facee6e046c4f3cafc6304e6dbaba4b3.js"></script>

As you may recall from my previous post, each aggregate maintains an internal collection of domain events.  This collection is implemented as `DomainEventCollection<TID>`, which is append-only.

<script src="https://gist.github.com/RyanMarcotte/eb0a2ebae1202185d12c782e2cfcb5d3.js"></script>

Finally, domain errors are also implemented as immutable classes.  They inherit from a `DomainError<TID>` base class or `DomainError<TParentID, TChildID>` base class.  The `DomainError<TID>` class is used for errors produced by the domain aggregate, while the `DomainError<TParentID, TChildID>` class is used for errors produced by domain entities owned by the aggregate.  Since entities are modified through commands executed against the aggregate, the errors produced by entities must inherit from `DomainError<TID>`.

<script src="https://gist.github.com/RyanMarcotte/da53548e571c694ba195f94576b1ad02.js"></script>

Now that we have discussed domain aggregates, domain entities, domain commands, and domain events, I can present the ReSharper code generation templates that I use: `domain_aggregate` ([link](https://gist.github.com/RyanMarcotte/4d638ba9495ecb4059c63c1b03fc673e)) and `domain_entity` ([link](https://gist.github.com/RyanMarcotte/5bd2ea7df18bdbbde3a788ef4a101d9d)).  Both templates create an initial shell for the domain aggregate or domain entity that includes a basic example of a domain command, domain event, and domain error.  After creating the shell, I organize the classes into the following folder structure.

``` text
Users
|-> Commands
|-> Errors
|-> Events
|-> Services
| UserId.cs
| User.cs
```

Note the "Services" folder...  We have not discussed *domain services* yet.  That's coming up next.
