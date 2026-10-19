# A budget is an email. Only a quota stops an agent from spending

One night in October, an agent on my team sent about 460,000 calls to a geocoding API from a batch job. It ran for five hours. Nobody was watching, because nothing needed watching: the cloud account had startup credits, and the working assumption was that the credits paid for everything on that account.

They did not. The credits covered the compute and the AI models on that provider. They did not cover the maps APIs, even though those APIs sit on the same project and the same billing account. The run cost about 2,000 euros of real money. We found out from a bank debit. The same evening, a different agent ran a web-scraping job past its monthly plan.

Both agents followed the rules they had. That is the point of this essay.

## The rule before

Before that night, our agents had detailed rules about irreversible actions. Sending an email, posting on a social network, updating the CRM, merging to the production branch, writing to a database: each of these is gated. In a supervised session, a human approves. In an unattended job, the action is denied unless the job's configuration allows it.

Spending money was not on that list. A paid API call looked like any other read: it returns data and can be retried. The guard rails classified actions by their effect on the outside world. A geocoding request has one effect only, on an invoice that arrives weeks later.

## What broke

Three assumptions failed at once.

The first was about coverage. "It should be covered by the credits" was true for most services on the account and false for the one that mattered. Credit programs have exclusions, and the exclusions are rarely the services you think about.

The second was about alerts. We had budgets. A cloud budget sends an email when spend crosses a threshold. It does not stop anything. And the console reports that a person checks lag by several hours, so a job that spends fast can finish before the first alert arrives. A report that shows zero net cost proves nothing while the credits absorb the visible part.

The third was about scale. A human who geocodes addresses by hand stops after a few hundred. An agent with a loop and a list of half a million rows does not get tired, and does not feel that the number is large.

## The rule after

We wrote one hard rule with six parts. It applies to every agent and every session.

1. Before any call to a usage-billed service, and always before a bulk run, the agent names the service, its unit price, the expected volume, and the expected amount outside credits. If the service is not covered by a credit that someone has verified, or if the amount is above 50 euros, the agent asks before it starts. "It should be covered" does not count as verified.
2. During the run, a hard cap bounds the spend. That means a daily quota on the API, or a call limit in the code. The geocoding and places APIs now have a quota of 1,000 calls per day. Agents may not raise it without an explicit request.
3. The cost check reads the detailed billing export. Console reports and estimates computed from usage do not count.
4. A budget is treated as what it is: an email.
5. Every progress report on a cloud run says in its first line who pays, credits or invoice, and on what evidence. Without evidence, the line says "coverage not verified".
6. To cut spend, cut at the source: the quota, the key, or the job. Never disable a shared API that other sessions use at the same moment.

The sixth part protects the other agents. Turning off a whole API to stop one job also stops every session that depends on it.

## What you can copy

If agents on your team can call paid APIs, you probably have the same gap. Most agent guard rails were designed around actions that change the world. Spend changes only the invoice, so it passes through.

Four things are cheap to do this week:

- List every usage-billed API your agents can reach, and write down for each one who pays. Check the exclusions of your credit programs line by line. Do not trust the account structure to tell you.
- Put a quota on each of them. Pick a number that a normal day never reaches. A quota is the only control that acts in seconds, without a human.
- Turn on the detailed billing export, and point your cost checks at it. Console summaries lag by hours.
- Add spend to the list of gated actions. Before a bulk run, the agent answers three questions: who pays, how much, and what stops it.

I cannot yet say how many incidents the rule has prevented. It is one week old. My hypothesis is that the costly failures of unattended agents look like this one. The agent does nothing visible. It does something ordinary half a million times.
