---
layout: single
title: "Implementing Domain-Driven Design: Domain Services (Pure)"
date: '2023-05-28 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
excerpt: Handling domain operations that span multiple domain entities and do not require information outside domain scope
---

[*Domain services*](https://enterprisecraftsmanship.com/posts/domain-vs-application-services/) encapsulate domain knowledge that do not naturally fit within domain entities and/or value objects.  This makes them especially useful for constructing workflows that affect multiple domain entities.

In Vladimir's article about [domain model purity versus domain model completeness](https://enterprisecraftsmanship.com/posts/domain-model-purity-completeness), he asserts the following trilemma:

>You can’t have all three of the following attributes:
>
>- **Domain model completeness**: when all the application’s domain logic is located in the domain layer; that is, not fragmented
>- **Domain model purity**: when the domain layer doesn’t have out-of-process dependencies
>- **Performance**, which is defined by the presence of unnecessary calls to out-of-process dependencies.

Much like the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem), you can only choose two of the three options:

>- **Push all external reads and writes to the edges of a business operation**: Preserves domain model completeness and purity but concedes performance.
>- **Inject out-of-process dependencies into the domain model**: Keeps performance and domain model completeness, but at the expense of domain model purity.
>- **Split the decision-making process between the domain layer and controllers**: Helps with both performance and domain model purity but concedes completeness.  With this approach, you need to introduce decision-making points (business logic) in the controller.

He ends the article by encouraging prioritization of domain model completeness and purity.  The next best option is to choose domain model purity over completeness.  This article illustrates an implementation of a domain service that maintains domain model completeness and purity.

First, a review of the `User` domain aggregate we have been using as an example thus far:

<script src="https://gist.github.com/RyanMarcotte/860889afb5ca2f4d899f795e76c2eab5.js"></script>

Note the `IsAccountOwner` property.  We have a few business rules surrounding its valid values:

- upon creating an organization in our application, an initial user account is created and is designated as the account owner
- an account always has an owner and can only ever have one owner
- ownership can be transferred from the current owner to another user, but only by the current owner and only if the other user is active

Clearly the second and third business rules cannot be enforced within a single domain entity.  We require the use of a domain service to encapsulate those rules.  That being said, we still need to add some supporting functionality to the `User` aggregate.

<script src="https://gist.github.com/RyanMarcotte/312bb770bccb80cd647aa2db992f2525.js"></script>

The two methods are declared `protected internal` because we only want our new domain service to be able to invoke these two methods.  Declaring them `public` would allow other code in the application outside the domain layer to invoke the methods like so:

- Invoking `RevokeAccountOwnership` on the current owner without invoking `GrantAccountOwnership` on a different user.  This "orphans" the account.
- Invoking `GrantAccountOwnership` on a different user and neglecting to invoke `RevokeAccountOwnership` on the current owner.  This designates two users as the account owner.

The above two scenarios obviously violate the business rule "an account always has an owner and can only ever have one owner".

When not using `abstract` domain aggregate classes, the two methods would be declared `internal` instead.
{: .notice--info}

The domain service implementation is straightforward.  It is just a concrete `public` class that accepts three `User` domain entities as dependencies: the "acting user" (the user performing the action), the "current admin" (the user that account ownership will be transferred from), and the "proposed admin" (the user that account ownership will be transferred to).  No interface is required.

<script src="https://gist.github.com/RyanMarcotte/9dcc17e97451262c6d5fbaeb037b9357.js"></script>

Any application code that needs to transfer ownership of the account must do so through this domain service.  This protects all the business rules.

Next week we will look at a different kind of domain service...  one that handles scenarios where information outside of the domain's scope is required.
