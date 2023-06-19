---
layout: single
title: "Implementing Domain-Driven Design: Domain Proxies (Simple)"
date: '2023-06-18 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
excerpt: Defining object-relational mapping for complex domain aggregates
---

In an [earlier post]({{ site.baseurl }}{% post_url 2023-05-14-domain-aggregates-and-domain-entities %}), we covered domain aggregates and how they are used to encapsulate data and behavior such that data is kept in a valid, consistent state at all times.  Recall that the example domain aggregate implementation used `abstract` classes.  In this post, I will explain the rationale for this implementation choice.

[*Object-relational mappers*](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) (ORMs) - such as [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/) and [NHibernate](https://nhibernate.info/) - are popular tools used to handle how application data is persisted and retrieved from a database.  While this technology is indeed helpful for handling serialization and deserialization of data, some of the data mapping technologies employed are quite complicated for complex data types and can often seem like a magic black box to more junior developers.  This represents a risk to application development and maintainability when domain-driven design is used to model the business domain, because the number of complex types requiring custom mapping is high.  This problem is compounded when your application is locked to using a legacy ORM - like [Entity Framework](https://learn.microsoft.com/en-us/aspnet/entity-framework) - because the legacy ORM is likely not as fully-featured as newer ORMs.

Rather than delegating the complex data mapping work to the ORM, you can instead use primitive types for your database models and then introduce *domain proxies* that handle how that primitive data is mapped to domain aggregates, domain entities, and value objects.  The name "domain proxy" is derived from the [*proxy* design pattern](https://refactoring.guru/design-patterns/proxy).  In our case, the domain proxy implements the `abstract` domain aggregate class.

- the `abstract` properties are implemented in the proxy, retrieving and mutating the underlying database model as necessary
- the `virtual` methods (i.e. domain commands) are overridden to perform database-specific actions before and/or after executing the domain command implemented by the aggregate

<script src="https://gist.github.com/RyanMarcotte/860889afb5ca2f4d899f795e76c2eab5.js"></script>

<script src="https://gist.github.com/RyanMarcotte/b949195d11fa0958bb9f28e04a27ba0c.js"></script>

The example domain proxy handles data mapping for the `Email` type and augments the domain aggregate with auditing functionality.  It is possible to perform both tasks using ORMs, but I find this implementation cleaner at the expense of having to write some boilerplate code.  The value of this particular approach may not be immediately clear given the simplicity of this particular domain aggregate, so in the next post I will discuss a more complicated example involving a domain aggregate that implements a finite state machine.
