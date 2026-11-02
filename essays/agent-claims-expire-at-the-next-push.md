# An agent's claim about a pull request expires at the next push

Last week, the agent that reviews pull requests on my team approved four of them while their CI was still running. On three others, it flagged a failing check as important and wrote that a re-run should clear it. The check failed on every pull request in the repository, because a secret it needed was empty. A re-run would have changed nothing. The backend owner had to correct the review by hand.

The same week, three reviewers flagged the same defect on twelve pull requests: the description no longer matched the code. An agent had pushed a fix in answer to their comments, and the test plan, the counts and the list of closed issues in the description still described the code from before the fix.

None of these counts came from a dashboard. They came from a weekly job where an agent reads the past seven days of agent sessions and pull request threads, looks for patterns that repeat, and edits the rules itself, with evidence attached to each change. I can veto a change after the fact. This week it changed two rules, and both are about the same thing.

## The rule before

Our agents write a lot of statements about state. A review says the CI is green. A pull request description says clippy is clean, lists what was tested and names the issues it closes. A follow-up comment says the requested change is done.

The rules told agents what to check before they wrote these statements. Run the tests before you claim they pass. Read the CI before you approve. Those rules were followed. The review agent did read the CI. It read it at a moment when some checks had not finished, and it treated "not failed yet" as "passed".

The description problem has the same shape. The description was accurate when the agent that opened the pull request wrote it. Then a second agent pushed a fix, and nothing in its instructions said that the push could make the old text false. It did its job on the code and left the text alone.

## What broke

Every statement an agent writes about a pull request is a snapshot. It is true at the commit and the moment it was written. Two things make it false without anyone lying:

1. Time. A check that is running will finish, and its result may differ from what you guessed.
2. A push. Any new commit can invalidate a test plan, a count, a line that says "no warnings", or a closing reference.

Our rules treated statements as facts to verify once. They did not treat them as claims with an expiry date. The prediction "a re-run should clear it" is the extreme case: a statement about a future state that the agent had not observed at all.

Humans make the same mistake, but a human reviewer usually remembers that the CI was still yellow and comes back later. An agent run ends. Nothing brings it back, so whatever it wrote stays on the page as if it were current.

## The rule after

Three changes, all written into the commands the agents follow.

First, the review agent waits for checks that are still running, once, with a hard limit of nine minutes. If they are still running after that, the review says so in one line and must not state anywhere that the CI is green.

Second, the review agent never predicts that a re-run will fix a failure unless it has started the re-run and seen it pass. An infrastructure failure unrelated to the diff goes at the end of the review as a single line. It never sits at the top as an important finding.

Third, after the agent that answers review comments pushes a fix, it reads the live description against the new diff and corrects only the lines that became false. It does not regenerate the description from a template, because that would erase what humans added. It names the correction in its follow-up comment. If nothing became false, it says that too.

I have not measured the effect yet. The weekly job will count the same patterns next Monday, and that count is the test.

## What you can copy

If your agents write about pull requests, sort their statements into three kinds and give each kind a rule.

- Observed now: a check that finished, a test that ran in this session. Write it, with the commit it applies to.
- Not finished: a check still running. Wait a bounded time, then write "still running". Never round it up to green.
- Predicted: anything about what a re-run, a merge or a deploy will do. Do not write it, or run the thing first and turn it into an observation.

Then add one rule for every agent that pushes: after a push, reread every statement your team relies on in that pull request, and fix the ones the push made false. The agent that changed the code is the only one that knows which lines it invalidated.

The cheapest place to find these failures was the reviewers' own comments. When three people complain about the same stale text on twelve pull requests, the rule is missing.
