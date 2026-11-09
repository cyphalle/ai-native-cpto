# If the agent can check it, the agent checks it

Last week I corrected my coding agents 20 times, across 12 separate sessions, for the same behaviour. Each time, the agent had finished a piece of work and then handed me a verification it could have done itself. "Please check that the pull request merged." "Can you confirm the other session is done?" "Have a look at the rendering and tell me if it works."

My answers were short. "Can't you just talk to the other session directly?" "Well yes, you need to look." "The agent never saw the rendering."

The count comes from a weekly job. An agent reads the past seven days of agent sessions, flags each human turn that corrects the agent, groups the corrections that repeat, and edits the operating rules with the evidence attached. This week, one group stood out by size.

## The rule before

Our agents run with a default of autonomy. They decide, act, and report the choice they made in one line. They ask at most one question per run, and only when the answer would change the deliverable. The checkpoint comes after the work, when I review what was delivered.

That rule was written to stop agents from asking permission. It said nothing about verification, and verification is where the agents kept stopping.

The pattern always had the same shape. The agent did the work. Then, at the edge of the task, it met a fact it had not confirmed: the state of a pull request, the end of a parallel session, a number shown in the product, a version in a changelog, what a slide or a chart looks like once rendered. And instead of confirming the fact, it wrote a sentence that handed the check to me.

## What broke

Every one of those 20 checks was within reach of the agent's own tools. The state of a pull request is one command away. Another agent session on the same machine can receive a message directly; the agent asked me to copy a text from one window to the other instead. A rendered document can be opened and inspected as an image, in light mode and in dark mode.

So the handoff was not caused by a missing capability. My reading is that it was caused by where the agent drew the boundary of its task. It treated "write the thing" as the task and "confirm the thing is true" as review, and review belonged to the human. Our own autonomy rule reinforced this by accident: it told the agent the checkpoint was mine and came after delivery. The agent placed the verification on my side of that line.

The cost of that boundary is not the time each check takes. The cost is the round trip. I have to notice the request, switch context, run the check, and come back to the session to report a result the agent could have read itself. At 20 a week, the human becomes the slowest tool in the loop.

There is a second cost, which I have not measured, so I state it as a hypothesis. When an agent ends a task with "please check", I cannot tell whether the work is done or only probably done. Only a verified report lets me stop thinking about the task.

## The rule after

The rule we added is one sentence long: what your tools can verify, you verify before you hand back.

It comes with a list of the cases that triggered it, because a list is easier for an agent to apply than a principle:

- the state of a pull request: read it yourself;
- the end of another session: message that session directly;
- a number in the product, a version in a changelog: look it up;
- a visual deliverable: open it, in light and dark, and describe what you see.

And it has one exception. "Please check" remains valid for an access the agent does not have, such as a device it cannot reach or an account it cannot log into. In that case the request must come with the steps, so the human can do it without thinking.

The same weekly pass added a second rule, from a smaller group of 9 corrections in 7 sessions. When I say "just do X", the agent does X, and proposes any safeguard, measure or process in one line afterwards. Both rules move work from the human back to the agent.

## What you can copy

If you run agents with a default of autonomy, three things transfer.

First, write verification into the definition of done. An autonomy rule that says "act, then report" will be read as "act, then let the human confirm" unless it says otherwise. Say that confirming the result is part of the task.

Second, enumerate the checks. Name the concrete states your agents keep handing back, the ones in your own logs, and write them into the rule. Agents generalise better from five named cases than from one abstract principle.

Third, count the corrections. We only saw this pattern because a job reads every session and groups what the human keeps repeating. One correction looks like noise. Twenty in a week is a rule that is missing.
