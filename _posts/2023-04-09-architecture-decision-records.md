---
title: Architecture Decision Records
date: '2023-04-09 12:47:36 -0600'
categories:
  - meta
tags:
  - documentation
  - architecture
excerpt: How I document architectural designs for myself and others
---

If you are not already familiar with *architecture decision records* (ADRs), please read [Michael Nygard's introductory article](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).  Amazon also has [prescriptive guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html) if you want more in-depth information.

Having worked for both small and large companies, I have found it incredibly important that development teams facilitate knowledge transfer both within their own team and with other teams.  I have also found that many developers are averse to writing documentation...  which is strange, given that we write for a living, technical writing skills notwithstanding.

ADRs are a lightweight way to document important decisions.  Through this practice, knowledge continuity is ensured.  A developer new to the code - or perhaps yourself six months from now - can read ADRs to gain insight into why an application or system was built the way it was instead of blindly accepting the current state of affairs and/or blindly changing the software without context.  A developer that changes teams or leaves the company but has written ADRs beforehand leaves a paper trail of accumulated knowledge.

Much like code, ADRs are read more often than they are written.  ADRs serve as a means of accountability for autonomous teams and help aid the development of a distributed architecture practice.  The effort to write clear, descriptive ADRs is an investment in your team's - and your company's - continued success.

The ADR template I use for my own work is as follows:

``` markdown
# 013. Use X for Y

## Status

What is the status, such as *proposed*, *accepted*, *rejected*, *superseded*, etc.?  If *superseded* by a subsequent decision, link to the subsequent decision.  Include the date that the status change was made.

## Context

What is the issue that we're seeing that is motivating this decision or change?  Are there any sociotechnical and/or budgetary concerns that must be factored into the decision?  Hyperlinks to supporting documentation and metrics captured at time of writing are encouraged.

## Options Considered

What options are available to solve the aforementioned issue?  What are the tradeoffs associated with each option?  Hyperlinks to supporting documentation are encouraged.

## Decision

What is the change that we're proposing and/or doing to solve the aforementioned issue?

## Consequences

What becomes easier or more difficult to do because of this change?  What are the immediate action items?

## References / Additional Reading

What individuals, teams, vendor documentation, and/or articles were consulted when gathering information throughout the decision-making process?
```

You are not required to write an entire ADR in one go; in fact, you are encouraged to solicit consultations and feedback throughout the writing process.

A catalog of other ADR templates can be found on [GitHub](https://github.com/joelparkerhenderson/architecture-decision-record#adr-example-templates), though most adhere to the same general format as above.  [Markdown Any Decision Records](https://adr.github.io/madr/) - itself derived from the usage of ADRs - use a similar format but increase the scope of records written to include any significant decision, architectural or otherwise.  The exact format of the document is not as important as the depth and clarity of information captured within the document itself and the value of that information to current and future readers.

Do not assume your readers can derive historical context or have the same knowledge base you do.  Some additional guidelines for writing ADRs:

- The first use of an abbreviation in the document should be accompanied by the full phrase for clarity; for example, "*architecture decision records* (ADRs)" or "Zone Redundant Storage (ZRS)".
- Be liberal with the use of hyperlinks to supporting documentation throughout the ADR.  Hyperlinks need not be exclusively reserved for the references / additional reading section.
- Avoid the use of raw hyperlinks; for example, opt for `[Markdown Any Decision Records](https://adr.github.io/madr/)` instead of `https://adr.github.io/madr/`.  This can be enforced with a markdown linter.
