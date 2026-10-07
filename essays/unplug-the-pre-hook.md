# Unplug the pre-hook: why my guardrails repair after instead of blocking before

Two rules matter more than most in our process. Every pull request leaves with an assignee and at least one reviewer. Every issue leaves with a priority and exactly one assignee. Without them, a pull request enters nobody's queue, and an issue cannot be ranked by anyone.

Agents forget these rules. So I built guardrails for them, and the first version was the obvious one.

## The first version: refuse before

Coding agents support hooks: scripts that run before or after each tool call. A pre-hook reads the command the agent is about to run and can refuse it. I wrote one for pull requests and one for issues. If the command to create a pull request had no reviewer flag, the hook refused it and told the agent why. Same for an issue without its priority section.

It worked, in the narrow sense. No malformed pull request got through.

Then I measured it. In five working days, the issue pre-hook blocked 35 commands. It was by far the first source of friction in my sessions. And when I looked at what each block produced, the answer was the same every time: the agent read the refusal, added the missing flag, and ran the same command again.

The block caught nothing that the second attempt did not fix. It cost a round trip, tokens, and my attention each time a session stalled on it.

## What a block really buys

A pre-hook makes sense when the action cannot be undone. If an agent is about to send an email to a customer, you must stop it before. There is no after.

Creating a pull request is the opposite case. A pull request without a reviewer is a temporary state that one command repairs in three seconds. The cost of a wrong state for a few seconds is close to zero. The cost of a refusal is a full round trip through the agent.

So the question to ask of any guardrail is simple. Can the wrong state be repaired after the fact, cheaply and with certainty? If yes, check after. If no, block before.

## The second version: check after, repair

The post-hook runs after the command. It does not trust the command text. It reads the real state of the pull request or the issue through the API: who is assigned, who is a reviewer, what priority is set.

This has three advantages over the pre-hook.

1. It covers every path. An agent can create a pull request through the command line, through an API integration, or by opening the browser. A pre-hook that parses one command misses the others. A post-hook that reads the final state sees all of them.
2. It repairs, and the work goes on. When the issue body carries the priority but the field is empty, the post-hook sets the field itself. When an issue has two assignees, it tells the agent which one to remove.
3. It reads truth. A command can look right and still fail silently. One of our automation workflows reports green even when it skips the step that sets the priority. The post-hook does not care what the logs say. It checks the field.

## Unplugging it

On 2026-09-14 I removed both pre-hooks from the configuration. The files stayed on disk, so the decision is cheap to reverse. The rules did not move: a pull request still leaves with a reviewer and an assignee, and an issue still leaves with a priority and one owner. Only the enforcement point moved.

Friction from these rules dropped to zero, and the share of well-formed pull requests and issues did not go down.

## The general lesson

Many guardrails for agents are built like guardrails for humans: refuse first, explain why. That design assumes that a mistake is expensive and that the actor learns from the refusal. Agents do not learn from a refusal across sessions, and many of their mistakes are cheap to fix.

So I sort every guardrail into two kinds.

- Irreversible actions: an email, a public post, a write to the production database, a change in the CRM. These are blocked before, and a human approves them. If no human is present, they are denied.
- Repairable states: labels, assignees, reviewers, fields, formatting. These are checked after, against the real state, and repaired.

The first kind protects the company. The second kind protects the process. Mixing them up gives you a process that is safe on paper and that nobody can afford to run. A guardrail that costs too much ends up removed, and then it protects nothing.
