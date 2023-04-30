---
layout: single
title: "DDD Fundamentals: Using Strongly-Typed IDs Instead of Primitive Types"
date: '2023-04-30 11:32:59 -0600'
categories:
  - domain-driven design
tags:
  - domain-driven design
  - types
excerpt: Improve code semantics by using classes instead of primitive types
---

Primitive types - `int`, `string`, `decimal`, etc. - are fundamental to software development in any programming language.  We use them to build up complex models of whatever business problems we need to solve.  While primitive types have no feature development overhead and are often required for easy serialization and deserialization of data, using them for workflow processing has some significant tradeoffs that negatively affect maintenance costs.  The use of primitive types for workflow processing is a design smell known as [*primitive obsession*](https://blog.ploeh.dk/2011/05/25/DesignSmellPrimitiveObsession/).  The canonical examples of primitive obsession tend to be phone numbers and ZIP codes: all phone numbers and ZIP codes are strings, but not all strings are phone numbers or ZIP codes.  We can avoid primitive obsession when working with database-generated integer IDs too.

- The type `int` has the range of values `(-2147483648, 2147483647)` but the range of valid IDs is `[0, 2147483647)`.  Depending on the ORM being used, `0` is often a special value indicating that the domain entity has yet to be persisted in the database.  When using `int`, validation rules to enforce the constrained range must be duplicated everywhere.
- Developers must take care to ensure that IDs are passed into functions correctly.  If two IDs are of type `int` then it is possible to mistakenly pass the wrong ID into a function; for example, `int materialId, int warehouseId` versus `int warehouseId, int materialId`.
- If a dictionary uses a primitive ID type for its key, then semantic information about the ID is not kept by the dictionary type; for example, `IReadOnlyDictionary<int, IReadOnlyCollection<MaterialInventory>>` versus `IReadOnlyDictionary<MaterialId, IReadOnlyCollection<MaterialInventory>>` for a dictionary containing collections of inventory for individual materials).  Developers can only rely on the accuracy of variable names in this scenario.

I have a pair of ReSharper code generator templates for quickly creating new ID types: one for *domain entity reference IDs* (`domain_entity_reference_id_int`) and one for *domain entity IDs* (`domain_entity_id_int`).  A *domain entity reference ID* represents an ID previously assigned to a domain entity; that is, the underlying integer is always greater than zero and represents a foreign key relationship.  A *domain entity ID* is owned by one domain entity and is either "new" or "existing"; that is, the underlying integer is either zero or greater.  Recall that an ID of zero means the domain entity has not been saved to the database yet.

<script src="https://gist.github.com/RyanMarcotte/25356c9f90600ec5ea0b455d343a42c8.js"></script>

<script src="https://gist.github.com/RyanMarcotte/480f5411fe3e89bfd2f3d9dd532c9310.js"></script>

If I want to create a new ID, I only need to start typing `domain...` and the templates will pop up.  Easy!  The generated code takes care of the boilerplate I would otherwise have to write for Entity Framework object-relational mapping to work.  Conversion between the two ID types is also handled.  I have yet to modify the templates since they were originally written.

Introducing ID classes works just as well if the underlying identifier is a `string` or `Guid`.
{: .notice--info}

The use of these ID classes improves code semantics.  In the example below, we can derive that `MaterialInventory` exists for a given `Material` and `Warehouse` that have been previously created, whereas that business rule would not be explicit and not enforced at compile time if `int` was used instead.

<script src="https://gist.github.com/RyanMarcotte/187fdafa58b5d664a12265ad27abd8de.js"></script>

Since initially introducing this technique, my teammates and I have noticed a dramatic improvement in code readability.  We organically replaced `int` IDs with these ID classes as we developed features and fixed bugs rather than attempting to replace all `int` IDs all at once.  I encourage you to employ this technique in your own applications.
