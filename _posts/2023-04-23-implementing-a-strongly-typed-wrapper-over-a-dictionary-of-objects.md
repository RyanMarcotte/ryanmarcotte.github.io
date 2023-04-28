---
layout: single
title: Implementing a Strongly-Typed Wrapper Over a Dictionary of Objects
date: '2023-04-23 16:17:31 -0600'
excerpt: A method for working with `Dictionary<string, object>`
---

Sometimes you need to store a dictionary of objects of different types by name; however, `Dictionary<string, object>` has no compile-time safety.  Each item retrieved from the dictionary requires that developers manually check the type of the value at runtime to ensure the type is expected.  It is possible that the code that adds items to the dictionary and the code that retrieves items from the dictionary are in different parts of the application, in which case using `Dictionary<string, object` directly increases the likelihood of mistakes and runtime errors due to mismatched types.

What we need is a wrapper over `Dictionary<string, object>` that enforces types at compile time: `TypedKeyDictionary`.  First, we must define `TypedKey`, which encapsulates a `string` and `Type`.

<script src="https://gist.github.com/RyanMarcotte/d745b3d8ab86cb3f0c9be017364e3b29.js"></script>

Instances of `TypedKey<T>` are then used to add, retrieve and remove items from `TypedKeyDictionary`.  

<script src="https://gist.github.com/RyanMarcotte/063a225f7231a5e622d3cfdfcf7bc539.js"></script>

**The above implementation is not thread-safe**.  You must modify `TypedKeyDictionary` to use `ConcurrentDictionary<TKey, TValue>` instead if thread-safety is a requirement for your application.
{: .notice--warning}

I typically define `TypedKey<T>` instances in a static class as `static readonly` properties.  This way the various `TypedKey<T>` instances can be reused where appropriate.

The approach described above can be slightly modified for use with `Dictionary<string, byte[]>` and `Dictionary<string, string>`; in these scenarios, the value stored in the dictionary has been serialized to either `byte[]` or `string`.

<script src="https://gist.github.com/RyanMarcotte/ddb65682d60c6d6eb7c474ad12821317.js"></script>

<script src="https://gist.github.com/RyanMarcotte/b3bae7a0d494acf751dc27081b3e05d5.js"></script>

If you are using Entity Framework or another ORM, we can use the same approach for data stored in a database too.

<script src="https://gist.github.com/RyanMarcotte/06163644054f4aae5fca27e382ff339d.js"></script>

As you can see, the basic idea has a variety of applications depending on your needs.  If you need to distinguish between different kinds of `TypedKey`, name them differently; for example, `CompanySettingKey` and `UserSettingKey`.
