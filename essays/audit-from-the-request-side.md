# An agent that answers code review needs an audit that starts from the review

In fourteen days, reviewers left 1,004 requests on my pull requests. An automated agent was in charge of answering them. When I audited that agent by hand, 16 requests had been dropped silently by one of its runs, and 32 more had been merged with no run and no commit at all. That is 48 requests, a bit under 5%, that a reviewer asked for and never got.

None of my monitoring had seen it. Every run had started, finished and logged a result.

## The setup

A scheduled job watches my open pull requests around the clock on a dedicated machine. When a human comment arrives after the last code push, it starts an agent that reads the thread, changes the code, runs the tests and pushes.

My monitoring answered one question: did the job run? A daily check read the logs and told me if a run had crashed, hung, or not started.

## What the audit found

The audit asked a different question. For each human request on a pull request, what handled it? A commit from the job, a commit from a human, a written reply, a follow-up issue. Anything with none of these is lost.

The losses had causes, and each cause was a place where the job recorded its own state and trusted it.

- **The cutoff moved by itself.** The job treats a comment as handled when a code push comes after it. Comments posted while a run was in progress were then hidden by that run's own push, which had never read them. Merging the base branch into the pull request had the same effect, and marked every earlier comment as answered.
- A failed attempt was stored as an empty one. When a run could not take the branch because another session held it, the job stored "tried, nothing to do". One pull request was skipped six times this way. In an earlier incident, a run started its tests in the background, ended its turn waiting for a notification that never came, and stopped with five changed files on disk. The job stored the same verdict and skipped that pull request 21 times.
- The agent read a review its own way. It reported one line per comment, so a comment with five numbered points could come back with three answered. It treated tests listed in a collapsed block as observations instead of requests. And it refused to touch any migration, including a migration the pull request itself had introduced, so those fixes reached the merge open.
- Approval ended the conversation. Most of the 32 merged requests were merged within minutes of an approval. The fastest one was merged 51 seconds after the comment.

## The rule before, and the rule after

Before, a run was the unit of truth. If the job had run on a pull request since the last comment, the pull request was considered handled.

After, the request is the unit of truth, and the job's state is only a claim to check against it.

1. A weekly read-only audit lists every human request of the week and matches it to what handled it. It sends me a report, and I decide what becomes an issue. Each high-severity finding is checked against the current code before it reaches the report, because the agents that judge each request sometimes pick the wrong commit.
2. Every thirty minutes, the job sweeps my pull requests merged in the last day and sends one message for any comment newer than its last real fix.
3. The job records the time window of each run, so a push from a run can no longer answer a comment posted during it. A run that lost the branch is not stored. An abandoned run is retried once before the job gives up on that state.
4. A scheduled agent may not wait in the background. It runs its tests in the foreground or it does not run them.
5. The merge step treats each numbered point of a review, and each point the agent itself flagged as open, as a blocking request.

The audit became weekly on the day of the first pass. Whether these fixes close the holes is a hypothesis today. The next reports will measure it.

## What another team can copy

If you run an agent that acts on human requests, whether review comments, support tickets or bug reports, monitor two things separately.

- Liveness: did the agent run, and did it finish? This is what logs give you for free.
- Coverage: for each request, can you name the thing that handled it? This needs a join between the requests and the outcomes, starting from the requests.

Liveness alone looks healthy while coverage leaks, because every leak I found happened inside a run that reported success. The weak points are the same in every such loop: a cutoff that decides what is new, a cache that remembers what was tried, and the moment a human approves. Audit those three first.

The 48 requests it found had each been written by a reviewer who assumed someone would read it.
