# How a CPTO runs a product and engineering team with agents

I am the CPTO of a B2B SaaS company with five people on product and tech. Coding agents write most of our code. This repository describes the operating model that makes that volume safe, and the measures that shaped it.

The model fits on one page. Each rule below exists because a measure showed the previous rule was wrong.

## 1. Autonomy by default, the checkpoint after

An agent that asks permission to think costs more than an agent that makes a wrong call. So the default is autonomous: the agent picks the reasonable option, does the work, and states its choice in one line. I correct it after delivery. A correction on a finished diff takes minutes. A question before the work blocks the work until I read it.

The agent asks first in two cases only:

- An external action that no revert can undo: an email sent, a CRM stage changed, a payment, a write to the production database.
- The deletion of data or files the agent did not create.

## 2. Decide alone, let the owner rule on the diff

A development choice (a migration, a new table, a new dependency, an architecture change) is no longer a question asked before writing code. The agent decides, writes it, and the pull request states the choice and the option it set aside. The owner of that layer rules on the diff in review.

An unmerged migration costs nothing to throw away. A question asked upfront stalls the work. The guardrail stays where it belongs: nothing enters the integration branch without the owner's review.

## 3. Owners, and what autonomy does not cover

Every domain has one owner: backend, frontend and infrastructure, data, and the science of the methodology. Autonomy is about execution. It does not move ownership. The agent goes fast on how, and it never decides who decides. A choice that belongs to an owner is written as a proposal, with the alternatives and the name of the owner who signs.

## 4. Review, never consultation, for science and data

For choices about the science and the data, the channel is review. The agent makes the call, writes it down, and the pull request asks the owner to ratify or correct a written choice. A written choice is faster to judge than an open question, and "What do you think?" reopens the debate we wanted to avoid.

This rule needed a second correction, described in section 6.

## 5. Two surfaces of text

Before writing anything, the agent asks who reads it.

- **Text for a human**: an issue, a pull request, a thread comment, a chat message. Someone must decide after reading. Length is capped by the act asked of the reader: about 600 characters to ratify, 1,000 to decide, 400 to execute, 300 to inform. A thread comment caps at 500 characters. The first line states the act.
- Text for an agent: a spec, an implementation plan, project context. Here the rule is the opposite. Be generous. Write the edge cases, the table names, the payload shapes, and what was set aside and why. An agent has no intuition to fill a gap, so every unstated assumption becomes a decision it makes alone.

Both surfaces live in one artifact. The human summary sits on top. The agent context sits below, in a collapsed block.

## 6. Measure every rule

Rules drift from practice. So I measure them, and I change the rule when the measure says so.

- **Pull request process.** Over 60 days, 346 of 423 pull requests (82%) were opened from a free-form command, outside the command that runs the layer checks and the review agents. The fix was to make that command the only path.
- **Review requests.** A rule added a reviewer for data and method whenever a pull request listed such a choice. Reading those requests showed that most of them only applied a decision already taken. We split the rule in two: every pull request still lists its data and method choices, and the owner is a reviewer only when the diff decides or changes one. See [the essay](essays/a-reviewer-is-a-commitment.md).
- **Blocking hooks.** A hook that refused malformed commands before they ran blocked 35 times in five working days, for no gain. A hook that checks the result after the command and repairs it does the same job. We unplugged the first one. See [the essay](essays/unplug-the-pre-hook.md).
- **Noisy scopes.** Most guardrail interruptions came from internal actions that a revert can undo. Scopes are now split by who an action affects, as section 7 describes.
- **Reviewers on solo repos.** On a tooling repository that only I touch, a reviewer was added by rule, and the request reached someone with no reason to read the diff. The rule now exempts those repositories.

## 7. Guardrails keyed on the absence of a human

Every agent tool call gets a tier. Tier 1 runs freely. Tier 2 runs and is logged. Tier 3 is irreversible and locked. The lock depends on whether a human is present, and the nature of the action matters less than you would think.

Scopes are split by who an action affects. External actions (email, CRM, payments, production database writes) ask a human every time. Internal actions on our own repositories pass in a supervised session and are logged.

- In a supervised session (a real terminal in the process chain), an external tier-3 action asks for approval, and the first yes opens a short window on that scope.
- A scheduled job runs against an explicit allowlist. A scope absent from the list is denied.
- With no human and no allowlist, the action is denied.

In bypass mode the gate never relies on `ask`: it shows its own confirmation dialog, and with no screen attached it denies. Guardrails must fail closed. The hooks are in [agent-guardrails](https://github.com/cyphalle/agent-guardrails).

## What this repository holds

- This page: the operating model.
- [essays/](essays/): one essay per rule that changed, with the measure behind it.

## License

Text under [CC BY 4.0](LICENSE).
