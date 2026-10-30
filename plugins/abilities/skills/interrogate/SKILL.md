---
name: interrogate
description: "Adversaries try to break a change or a design, and only findings that a run proves survive."
disable-model-invocation: true
---

# Interrogate

Adversaries try to break a change or a design. A finding survives only when a run proves it. Report the findings. Do not apply fixes.

## Frame

The scope is what the user names. Otherwise it is the branch diff from its merge base with the base branch, through the working tree. Restate the intent in your own words, from the user's message, the commits, and the PR description. Ask the user when the intent is unclear.

## Fan out

1. Pin the code. This command makes one commit that holds the tracked and untracked changes. It does not change the index or the stash.

   ```sh
   snapshot=$(
     GIT_INDEX_FILE=$(mktemp -u) && export GIT_INDEX_FILE
     git read-tree HEAD && git add -A &&
       git commit-tree "$(git write-tree)" -p HEAD -m "interrogate snapshot"
   )
   echo "$snapshot"
   ```

2. Spawn three adversaries in one message, in the foreground, as `general-purpose` agents with `isolation: "worktree"`. Add one adversary for each further risk area in the change.
3. Give each adversary a different angle, because all adversaries use models from one vendor:
   - Inputs: empty, bad, large, and boundary values.
   - Timing: state, ordering, retries, and concurrency.
   - Consumers: callers, stored data, and formats outside the diff.
4. Give each adversary the scope, the intent, the snapshot commit, its angle, and this brief:

> First, run `git checkout --detach <snapshot>` in your worktree, and work only from that commit. Break this change from your angle. Prove each finding with a run against the real code, such as a failing test, a repro script, or command output that shows the wrong behavior. For a design or plan, run a prototype or probe the current tree. Keep repros in your own worktree. Report your worktree path. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run. That covers style, naming, structure, and risks that need a future change to matter. No findings is a valid result.

If an adversary produces no output, continue without it and report the dropout.

## Verify

Rerun each finding's command in that adversary's worktree. Keep a finding when the output shows the failure and the failure breaks the intent. Drop the rest. Merge findings that share a root cause.

The verdict keeps every verified finding from every adversary, because each adversary finds different failures.

## Output

- **Intent**: the restatement.
- **Findings**: for each verified finding, the command, its output, how it breaks the intent, and the adversaries that found it. Say "no verified findings" when none survive.
- **Dropped**: for each adversary, the count of dropped findings and why.
- **Worktrees**: the path of each adversary's worktree, so that the user can rerun a finding.
