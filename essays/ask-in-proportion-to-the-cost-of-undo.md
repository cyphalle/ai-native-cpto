# Ask for permission in proportion to the cost of undo

Last week, in 10 separate agent sessions, a coding agent stopped to ask me for permission and I answered with one or two words. "Go." "Merge it." "Ok, commit and push then." In none of the 10 did my answer change anything. The agent had already written the comment, checked the pull request, or finished the work. It only needed a word to release it.

One of those sessions shows the pattern well. The agent wrote a short analysis for a GitHub issue, then ended its turn with: "I did not post it on the issue, because a GitHub comment is an external action." My reply was "go". The comment went up a minute later, unchanged.

The count comes from a weekly job. An agent reads the past seven days of agent sessions, flags each human turn that corrects or unblocks the agent, groups the ones that repeat, and edits the operating rules with the evidence attached. This group had 10 occurrences in 10 different sessions, so each one was a separate stop.

## The rule before

Our agents run with a default of autonomy. They decide, act, and report the choice they made in one line. The checkpoint is after delivery, when I review what came out.

The default had two exceptions, and they were written as categories. An irreversible external action needs my approval: sending an email, a chat message or a social post, updating the CRM, merging into the production branch, deploying to production. Deleting data or files the agent did not create needs my approval too.

That list looked precise. It was not. The word doing the work was "external", and the agent read it as "visible to someone else".

## What broke

Under that reading, three kinds of action kept falling into the exception, and none of them belonged there.

A comment on an issue or a pull request is visible to the team. It is also edited or deleted in one click. Its cost of undo is close to zero. The agent treated it like an email to a customer, which cannot be recalled.

A merge into our integration branch is visible too. But we merge through a command that checks the approvals, the CI status and whether every review request in the thread was answered, and it refuses on its own when one condition is missing. My "go" added a human check on top of a machine check that already said yes. It caught nothing the command would have missed.

A commit and push in my own tooling repositories is visible to nobody but me. The agent still ended turns on "nothing is committed yet", waiting for a word. The work sat in a working tree until I came back.

The common mistake was to price the question by who can see the action. The useful price is what it costs to take the action back, and whether something else already refuses when it should.

## The rule after

We kept the exceptions and added an explicit list of what is not one. A comment on an issue or a pull request goes out without asking. Its length limits and its language rules still apply, so the content is constrained even when the permission step is gone. A merge into the integration branch through the merge command goes out without asking when the pull request is approved and green, because the command refuses by itself. A commit and push in my tooling repositories happens at the end of the work, and a turn there does not end on uncommitted changes.

Two details matter. A command whose own specification prescribes a confirmation keeps it, so a reviewing command run interactively still asks before posting. And the production branch, emails and customer-facing messages stay where they were. Drafts of messages to people still stop for my review, because a sent email has no undo.

I have not measured the effect yet. My expectation, stated as a hypothesis, is that the count of one-word approvals drops close to zero next week. The weekly job will tell me.

## What you can copy

Count the one-word approvals. Every "go" you type with no edit is a question that should not have been asked. It is the cheapest signal you have on where your agents draw the permission line, and it sits in your session logs already.

Price each permission by its cost of undo. A one-click undo needs no question. An action that cannot be recalled needs one every time.

Do not stack a human on top of a guard that already refuses. If a command checks the conditions and blocks when one fails, a human approval on the same conditions is a second copy of the same check, paid in attention.

Write the non-exceptions down by name. A model will generalize a category word like "external" further than you meant. A short list of named actions that do not need a question is harder to misread than a better definition of the category.
