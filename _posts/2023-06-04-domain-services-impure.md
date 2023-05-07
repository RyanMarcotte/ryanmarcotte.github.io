---
layout: single
title: "Implementing Domain-Driven Design: Domain Services (Impure)"
date: '2023-06-04 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
excerpt: Handling domain operations that require information outside domain scope
---

Recall that [*domain services*](https://enterprisecraftsmanship.com/posts/domain-vs-application-services/) encapsulate domain knowledge that do not naturally fit within domain entities and/or value objects.  In this post, we will consider another scenario where domain services are helpful: when data not held by the aggregate is required for the aggregate to make a decision.

We will use uniqueness checks as an example.  It is not feasible to load all data necessary to perform the uniqueness check into the aggregate because that data will be retrieved even for domain commands not requiring that data; instead, the required data will be loaded separately and then passed into the domain command.  We do not want to introduce infrastructure dependencies into our domain aggregates and so we require an interface to provide a seam between when the data is retrieved and when the data is used.  The interface will be defined in our *domain assembly* while the implementation of that interface will be defined in an *application service assembly*.

I avoid the use of `Task<T>` in domain assemblies as that is a strong code smell that infrastructure dependencies are bleeding into the domain aggregate.  The domain aggregate only acts on data that has already been loaded into memory.
{: .notice--warning}

The interface is straightforward.

<script src="https://gist.github.com/RyanMarcotte/2555b8adc995ef014b87b9332263fdd7.js"></script>

The `IDoThing` naming convention for domain service interfaces is a personal preference.  Feel free to substitute your own.
{: .notice--info}

The implementation of the domain service interface is also relatively simple.  It works on data that has been previously retrieved from the database: the collection of all user IDs and corresponding emails.  I would write a set of automated tests for this class to ensure it functions correctly.

<script src="https://gist.github.com/RyanMarcotte/64800ab5d60debd5de6384226eefedbe.js"></script>

As previously mentioned, the data will be passed into the domain command: the domain command parameter object accepts the proposed `Email` and an instance of the domain service.

<script src="https://gist.github.com/RyanMarcotte/af8e35c7978c679a59f0a6c39c530127.js"></script>

All that remains is using the domain service inside the aggregate.

<script src="https://gist.github.com/RyanMarcotte/38893b7a7785a7451cd73715fa803705.js"></script>

Earlier, I mentioned the terms *domain assembly* and *application service assembly* without defining them.  The definitions require an explanation of C# solution organization, so I will dive into those details next.
