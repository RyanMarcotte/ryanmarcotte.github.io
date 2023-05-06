---
layout: single
title: "Implementing Domain-Driven Design: Value Objects"
date: '2023-05-07 09:30:00 -0600'
category: domain-driven design
tags:
  - code
  - domain-driven design
excerpt: Encapsulating concepts from your business domain
---

[*Value objects*](https://martinfowler.com/bliki/ValueObject.html) are a foundational building block in domain-centric applications.  Value objects have two important properties:

- they have no identity...  instances of value objects are interchangeable with each other if their data is equivalent; that is, [structural equality](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons#value-equality) is used rather than [reference equality](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons#reference-equality)
- they are immutable...  once a value object instance is created, it cannot be changed

Examples of value objects could include:

- `CanadianProvinceCode`...  enforces [two-letter province codes](https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/completing-slips-summaries/financial-slips-summaries/return-investment-income-t5/provincial-territorial-codes.html) for Canadian provinces (`BC`, `AB`, `SK`, ...)
- `AmericanStateCode`...  enforces [two-letter state codes](https://www.faa.gov/air_traffic/publications/atpubs/cnt_html/appendix_a.html) for American states (`CA`, `NY`, `FL`, ...)
- `StateOrProvinceCode`...  discriminated union of `CanadianProvinceCode`, `AmericanStateCode`, or an optional free-text `string` for other countries
- `ISO3166CountryCode`...  enforces [two-letter codes defined by the ISO-3166 standard](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes) (`CA`, `US`, `MX`, ...)
- `CanadianPostalCode`...  enforces postal code format rules for Canada
- `AmericanZipCode`...  enforces ZIP code format rules for United States
- `PostalCode`...  discriminated union of `CanadianPostalCode`, `AmericanZipCode`, or an optional free-text `string` for other countries
- `Address`...  street, suite, city, `StateOrProvinceCode`, `ISO3166CountryCode`, `PostalCode`
- `ISO4127CurrencySymbol`...  enforces [three-letter codes defined by the ISO-4127 standard](https://www.xe.com/iso4217.php) (`CAD`, `USD`, `EUR`, ...)
- `MoneyAmount`...  quantity and `ISO4127CurrencySymbol` (`$5.00`, `€20`, ...) with support for conversion between currencies
- `PhoneNumber`...  enforces phone number syntax
- `Email`...  enforces email address syntax
- `MassMeasuredQuantity`...  quantity and unit of measurement (`g`, `mg`, `kg`, `oz`, `lb`) with support for equality, relational operators, and conversion between units
- `VolumeMeasuredQuantity`...  quantity and unit of measurement (`L`, `mL`, `hL`, `fl oz`, `gal`) with support for equality, relational operators, and conversion between units
- `MeasuredQuantity`...  discriminated union of `MassMeasuredQuantity`, `VolumeMeasuredQuantity`, or `decimal` for counts
- `ReferralCodeCandidate`...  accepts any string but defines a function that validates whether that string is a valid referral code for the application; that is, all strings are referral code candidates but only some strings are valid referral codes
- `ReferralCode`...  holds a valid referral code

The strongly-typed IDs introduced in my [previous post](2023-04-30-using-strongly-typed-ids-instead-of-primitive-types.md) are also examples of value objects.
{: .notice--info}

Note how value objects can be used to implement other value objects; for example, `Address` uses `StateOrProvinceCode`, `ISO3166CountryCode`, and `PostalCode`.

I have seen some applications introduce a `ValueObject` base class that value objects inherit from but I have personally found this approach to be overkill.  An immutable class that implements `IEquatable<T>` has been sufficient for me.  ReSharper (or other productivity tools) can then be used to generate the equality operations for you.
