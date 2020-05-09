In the [previous post](../recommendations-for-a-git-beginner/) in this series on evolving your use of `git`, I outlined how `git` can be used in a one-person project. 

In this post, I'll outline additional things you may need to learn about `git` to work effectively as part of a team.

<!-- TEASER_END -->

## Formalize `git` conventions for your team project
Obviously, one-person projects are easy to maintain. Once you have two or more people adding code to the same repository, everyone should follow common conventions for branch naming, tagging, coding and so forth. Every organization has its own standards or best practices, and many recommendations are freely available on the internet. What's important is to pick a suitable convention early on and follow it as a team.

Also, it's likely that different team members will have different levels of expertise with `git`. You should create and maintain a basic set of instructions for performing common `git` operations following the project's conventions.


## Merging changes
Although each team member works on a separate feature branch, everyone will eventually modify some common files. When merging the changes back into the `master` branch, the merge will typically not be automatic. Human intervention may be needed to reconcile the different changes made by two authors to the same file. This is where one has to learn to deal with `git` merge techniques.

Modern editors have features to help with `git` merge. They will clearly indicate various options for a merge in each part of a file, such as whether to keep your changes, the other branch's changes, or both. It may be time to pick a different code editor if yours doesn't support such capabilities.


## Rebase your feature branch often
As you continue to develop your feature branch, rebase it against `master` often. This means executing the following steps regularly:
```bash
git checkout master
git pull
git checkout feature-xyz  # name of your hypothetical feature branch
git rebase master  # may need to fix merge conflicts in feature-xyz
```

The above steps result in rewriting history in your feature branch (and that's not a bad thing). Your feature branch is first made to look like `master` with all the updates made to `master` up to this point. And then all your commits to the feature branch are replayed on top so that they appear sequentially in the `git` log. You may get merge conflicts that you'll need to resolve along the way which can be a challenge. However, this is the best point to deal with merge conflicts because it is only impacting your feature branch.

After you fix any conflicts and perform regression testing, if you're ready to merge your feature back into `master`, you'd perform the above rebase steps one last time and then perform the merge:
```bash
git checkout master
git pull
git merge feature-xyz
``` 

In the interim, if someone else had pushed changes to `master` that conflicts with yours, the `git` merge will again have conflicts which you'll need to resolve, and repeat any necessary regression testing.

There are other merge philosophies (e.g. without rebasing and only using merge to avoid rewriting history), some of which may even be simpler to use. However, I've found this approach to be a clean and reliable strategy. The commit history is stacked up as a sequence of features which is meaningful.

With "pure merge" strategies (without rebasing regularly as suggested above), the history in the `master` branch will be interspersed with the exact commit occurrences across all the features being developed concurrently. The exact commit times are not usually that important, and such a mixed up history is hard to review. 


## Squash commits before merging
When working on your feature branch, it's fine to add a commit for even minor changes. However, if every feature branch produced, say 50 commits, the resulting number of commits in the `master` branch could grow unnecessarily large as features get added. In general, there should only be one or a few commits added to `master` from each feature branch. To achieve this, you *squash* multiple commits into one or a handful of commits with more elaborate messages for each one. This is typically done using a command such as:
```bash
git rebase -i HEAD~20  # look at up to 20 commits to consider squashing
```

When the above is executed, an editor will pop up with a list of commits that you can act upon in several ways including *pick* or *squash*. Picking a commit means keeping that commit message. Squashing implies combining that commit's message into the previous commit. Using these and other options, you can combine commit messages into one, and perform some editing and clean up. It's also an opportunity to get rid of the commit messages that aren't really important (e.g. a commit message for fixing a typo).

Note that my suggestion here is to keep all the actions associated with the commits, but only combine the associated message text and edit them for improved clarity prior to merging into `master`. Be careful not to drop a commit which is also an option of `git rebase` unless you actually intended the drop.

After performing such a rebase, I like to look at the `git log` one last time to make any final edits with:
```bash
git commit --amend
```

Finally, a forced update to your remote feature branch is necessary since the `git` commit history for the branch has been rewritten:
```bash
git push -f
```


## Using tags
After you have performed the necessary testing and are ready for deployment of the software from the `master` branch, or if you want to preserve the current state as a significant milestone for any other reason, you should create a `git` tag. While a branch accumulates a history of changes corresponding to commits, a tag is a snapshot of the branch's state at that point in time. A tag can be thought of as a history-less branch or as a named pointer to a specific commit immediately after which the tag was created.

Configuration control is about preserving the state of code at various milestones. Being able to reproduce software source code for any milestone when necessary so that it can be rebuilt is a requirement in most projects. A `git` tag provides the unique identifier for such a code milestone. Tagging is straightforward:
```bash
git tag milestone-id -m "short message saying what this milestone is about"
git push --tags   # don't forget to explicitly push the tag to the remote
```

Consider the scenario where software corresponding to a given `git` tag is distributed to a customer, and the customer reports an issue. While the code in the repository may continue to evolve, it's often necessary to go back to the state of the code corresponding to the `git` tag to reproduce the customer issue precisely and create a bug fix. Sometimes the newer code may have already fixed the issue but not always. Typically, you'd checkout the specific tag and create a branch from that tag:
```bash
git checkout milestone-id        # checkout the tag that was distributed to the customer
git checkout -b new-branch-name  # create new branch to reproduce the bug
```

Beyond this, consider using annotated tags and signed tags if these may be beneficial to your project.

## Have the software executable print the tag
In most embedded projects, the resulting binary file that is created from a software build usually has a fixed name. The `git` tag corresponding to the software binary file cannot be inferred from its filename. It is useful to "embed the tag" into the software at build time so as to correlate any issues precisely to a given build. Embedding the tag can be automated as part of the build process. Typically, the tag string generated by `git describe` is inserted into the code before code compilation so that the resulting executable will print the tag string while booting up. When a customer reports an issue, they can be guided to send a copy of the boot output.

## Conclusion
My goal here was to provide some ideas on how a team can work successfully with `git` on more sophisticated projects. I hope you found some of these ideas useful.
