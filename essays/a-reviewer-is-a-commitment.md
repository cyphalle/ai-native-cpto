# A reviewer is a commitment: how we cut review requests to the data owner by tying them to decisions

In our team, being added as a reviewer means you answer within 24 hours. That makes a review request expensive for the person who receives it. For two weeks in September I did not treat it that way, and the numbers showed it.

## The rule that over-fired

Our product computes figures from scientific methods and reference data. Two people own that layer. The chief science officer owns the science: what we measure, which formula, which threshold. The data owner owns the implemented data: which table, which value, which source.

We had already learned not to ask them open questions before the work. Asked cold, "what do you think about this threshold?" often gets no strong answer, and the work waits. So the agent makes the call, writes it down, and the pull request asks them to ratify or correct a written choice.

To make that work, every pull request that touched a method or data choice carried a section listing those choices, one per line, with the option set aside. And the rule coupled that section with a reviewer: if the section exists, add the data owner as a reviewer.

## What the measure said

I measured the last 150 pull requests. In two weeks, the data owner received 19 review requests and answered 5.

Then I read the 19. Only one of them touched a reference table. The other 18 applied a decision that already existed: they displayed a value in a unit already chosen, aggregated the way the rest of the code aggregates, wired a source already selected, or fixed a bug in a calculation whose formula did not change.

At 19 requests for one real question, a person does the rational thing. They sort by ignoring. The one request that mattered drowned in the others.

## Two gestures, split apart

The mistake was to treat two separate acts as one.

The first act is to leave a trace. The section that lists method and data choices costs three lines. Six months later, it tells anyone why a figure looks the way it does. It calls nobody. It should stay wide: any formula, threshold, unit, source, reference table, aggregation rule or data scope gets a line.

The second act is to ask for a decision. A reviewer request puts a person on the hook for 24 hours. It should stay narrow: only when the diff makes a choice for the first time, or changes a value already in place.

So we split the rule. The section is always there when it applies. The data owner or the chief science officer is a reviewer only when the diff decides or changes something in their domain. When they are added, it is on top of the layer reviewer, and they ratify a written choice.

## Five cases that call neither

We wrote down the cases from September that should never have reached them:

1. A release pull request from the integration branch to production.
2. A label change or a style sheet.
3. An operations script that touches the database without changing a business value.
4. A schema migration or a uniqueness constraint. The shape of a table belongs to the backend owner. The data owner owns what is inside it.
5. A refactor that moves a calculation without changing it.

A short list of real examples beats an abstract rule. An agent reading "only when the diff decides" can still hesitate. An agent that sees "a uniqueness constraint does not count" does not.

## Why this matters more with agents

When humans open pull requests, the volume limits the damage of an over-firing rule. With agents, I open more than 500 pull requests a month. Any rule that adds a reviewer by default multiplies at that rate. A rule that sends one unneeded request a day to a human with a human's day is a rule that burns trust.

The general principle: a mention is a question, and a review request is a commitment. Write the question next to the name. If nothing is expected, write FYI, so the reader knows that skipping it is fine. And never ask someone to commit to a review just to keep them informed.
