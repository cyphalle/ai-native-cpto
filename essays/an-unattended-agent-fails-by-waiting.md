# An unattended agent fails by waiting, so make it fail fast

On a Monday evening in October, at 21:03:27, a scheduled agent run listed a folder in my Downloads directory. The command never returned. The run produced no error, no output and no alert. Someone killed it by hand the next day, 19 hours and 44 minutes later.

Over the following three days two more scheduled runs froze the same way. A health-check job stayed stuck for 2 days and 14 hours. A job that applies code review comments stayed stuck for 22 hours and 37 minutes. Three runs, more than four days of dead compute in total, and not a single line in any log that said something was wrong.

## The rule before

We run our agent jobs on a schedule, on a laptop that sits on a shelf with nobody in front of it. Each job is a headless coding agent with a prompt, a set of tools and a time slot. Until that week the operating rule was simple: give the agent a clear task, let it run, and read the summary it posts when it finishes.

That rule assumes a run ends. It either succeeds, or it fails with an error that someone reads. Our monitoring looked for both outcomes, and it was built on that assumption.

## What broke

macOS protects a few folders, Downloads, Documents and Desktop among them. The first time a program touches one of them, the system opens a permission dialog and the call blocks until a human answers. The permission is stored per binary path. Our agent CLI installs each new version under a new path, so every update turns it into a stranger that has to ask again.

With a person at the machine, this is a one-click annoyance. On a shelf, it is a trap. The dialog opens on a screen nobody looks at. The shell command waits. The agent waits for the shell. The scheduler sees a process that is alive and well. The system log held the request and no answer: the prompt was opened and never resolved.

None of the three runs needed those folders for its task. One had its test data there by habit. The other two ran a broad `ls` or `find` while exploring. That is ordinary agent behaviour, and you cannot predict every place an agent will look.

## The rule after

We added three layers, in the order the incidents taught them. The order we would recommend is the reverse.

The first was a watchdog on silence. It shipped the day after the first freeze. The job runner now kills any agent run that writes nothing to its session log for 30 minutes. A timeout on total runtime was the obvious choice, but healthy runs vary widely in length. Silence is the better signal. A working agent produces tool calls and output all the time, and a blocked one produces nothing.

The second was an instruction. The shared execution prompt that every scheduled job receives now says, in one line, never to read, list or search those three folders, because a dialog nobody answers freezes the run. It costs nothing. It is still a request to a model, and a model that explores will sometimes forget it.

The third was a sandbox. The runner now starts every agent under an operating system sandbox profile that denies the three folders outright. The denial happens before the permission system is consulted. We checked it on the shelf machine: a read of Downloads now fails with "Operation not permitted" in under a second, no permission request appears in the system log, and the agent reads that error like any other and moves on. A job that really needs a file reads it from a location outside the protected folders, set through an environment variable.

## What you can copy

The general lesson is that an unattended agent has a failure mode that attended software rarely shows. It does not crash. It waits for a human who is not there. Anything that can prompt is a candidate: an OS permission dialog, a CLI asking "are you sure?", a login page in a browser tool, a package manager asking to accept a licence. Your error monitoring will not see any of them.

Three things follow, and they are cheap:

1. Make the blocking path fail instead of wait. Find what can prompt on your agent hosts and deny it at the OS or tool level. A fast, explicit error is something the agent can recover from. A dialog is not.
2. Watch output, and treat elapsed time as a weaker signal. Kill and alert on a run that has been silent longer than your slowest healthy step. Our threshold is 30 minutes. Pick yours from your own logs.
3. Keep the prompt rule, and do not rely on it. An instruction lowers the rate of the mistake. A sandbox removes it. You want both, because the instruction also tells the agent why the read failed when it does.

The order matters. We started with the watchdog because it was the fastest to write, and it bounds the cost of every future hang. The sandbox is the layer that would have prevented all three incidents. If you run agents on a schedule, start there.
