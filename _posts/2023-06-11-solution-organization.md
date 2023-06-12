---
layout: single
title: "Implementing Domain-Driven Design: Solution Organization"
date: '2023-06-11 09:30:00 -0600'
category: domain-driven design
tags:
  - architecture
  - domain-driven design
excerpt: My preferred approach to organizing C# code across multiple projects
---

Kamil Grzybek has an excellent article series on *modular monoliths* that serves as the foundation for how I organize code.  I encourage you to read through his articles first if you have not already done so.  I'll wait.

- [Modular Monolith - Primer](https://www.kamilgrzybek.com/design/modular-monolith-primer/)
- [Modular Monolith - Architectural Drivers](https://www.kamilgrzybek.com/design/modular-monolith-architectural-drivers/)
- [Modular Monolith - Architecture Enforcement](https://www.kamilgrzybek.com/blog/posts/modular-monolith-architecture-enforcement)
- [Modular Monolith - Integration Styles](https://www.kamilgrzybek.com/design/modular-monolith-integration-styles/)
- [Modular Monolith - Code Example](https://github.com/kgrzybek/modular-monolith-with-ddd)

The code in the monolith is organized into *modules*.  Individual modules are cohesive groups of application functionality.  A module could be a fully-autonomous application in its own right because it is highly encapsulated.  Modules only communicate with each other asynchronously over a message bus; this ensures encapsulation of individual modules is preserved.  The set of modules is deployed as a single unit, forming the monolith.

The following project structure is my (current) preferred approach for organizing code.  `MODULE_NAME` is a placeholder; examples of module names include `Administration`, `Inventory`, `Production`, and `Transactions`, but the names obviously depend on what business domain you are operating in.  Some of the tactical patterns listed below have not yet been covered in my **Implementing Domain-Driven Design** article series, but will be in future posts.

| Project Name | Layer | Purpose | Tactical Patterns Used |
| --- | --- | --- | --- |
| `MODULE_NAME.Domain` | domain | Forms the core of the application, defining business entities and operations independently from infrastructure concerns. | domain aggregates, domain entities, domain entity IDs, value objects, domain commands, domain events, domain errors, domain services, domain factory parameters |
| `MODULE_NAME.Domain.Reference` | domain | Defines data that can be shared across domains, most often reference IDs. | domain entity reference IDs, value objects |
| `MODULE_NAME.Module` | application | Defines the commands and queries that can be performed by the module to perform business actions. | request handlers, service error factories, domain repository parameters, readonly database query handlers (as required) |
| `MODULE_NAME.Module.Integration` | application | Defines the business capabilities of the module that are usable by other modules through the use of integration commands and integration events. | integration command handlers, integration event handlers, readonly database query handlers (as required) |
| `MODULE_NAME.IntegrationCommands` | application | Defines the integration commands used to communicate between modules. | integration commands |
| `MODULE_NAME.IntegrationEvents` | application | Defines the integration events used to communicate between modules. | integration events |
| `MODULE_NAME.Database` | infrastructure | Implements database models and the mappings between those database models and objects defined by the domain layer. | domain proxies, domain factories, domain repositories |
| `MODULE_NAME.WebAPI` | hosting | The module's HTTP endpoint implementations - usually as [controllers](https://learn.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-7.0) - that will be invoked via HTTP. | |
| `MODULE_NAME.WebAPI.Bootstrapping` | composition root | The module's [*composition root*](https://blog.ploeh.dk/2011/07/28/CompositionRoot/).  This project takes dependencies on all other projects in the module. | |

I have received some feedback that the project structure may be too fine-grained, but I personally find it easier for helping to control dependencies; for example, `MODULE_NAME.Domain.Reference`, `MODULE_NAME.IntegrationCommands`, and `MODULE_NAME.IntegrationEvents` are used across module boundaries and are easy to do so because those projects have no dependencies of their own.  The project dependencies are depicted below.

![module project dependency diagram]({{site.baseurl}}/assets/images/module-project-dependencies.png)

The solution's startup project takes dependencies on all `MODULE_NAME.WebAPI.Bootstrapping` assemblies - effectively depending on everything - and invokes the applicable bootstrapping code for all modules.  How this bootstrapping is done will be explained in a future post after all other tactical patterns have been covered.
