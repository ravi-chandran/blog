I started using `git` several years ago. The sophisticated way in which `git` swaps files back and forth "under the hood" (inside the mysterious `.git` folder) when changing branches still amazes me to this day. In my first job years ago where I worked on signal processing research, none of the researchers used any formal version control systems. Every day, I would make a "just-in-case" backup of my code that was in a good working state before making further changes. Even after I started using `git`, there was an initial period of anxiety when I made such backups because I didn't really know all the ways of undoing changes with `git` and feared a mishap.

I wish I had known about the approaches and opportunities for code management that `git` provides beyond the high level explanations about its importance. While there is a staggering amount of documentation and tutorials on `git`, it is a sophisticated tool that is impossible to learn all at once while also trying to focus on the application you're developing.

This is not a complete tutorial on `git` but a sequence of stages to gently evolve your knowledge and use of `git`. Had I known these stages beforehand, I would have stumbled less along my journey with `git`. In this first article, I'm going to outline how to evolve your usage of `git` in a one-person project.

<!-- TEASER_END -->

## Starting use of `git`
In any reasonably complex software project, writing code involves adding files, making changes to files, undoing changes, sometimes redoing changes, comparing files between branches, and making sure you don't lose any work inadvertently along the way.

If you're worried that you need to completely understand how commits get stacked up, how merges and rebasing works, and the nuances of manipulating the `HEAD` pointer, well, you don't really need to know these things to get started using `git`.

## Using `git` as a backup mechanism
A first step is to start using `git` for projects where you're the only author. It could even be for your own personal notes or documentation, not necessarily software. (I use `git` for my entire blog.) At this stage, `git` is primarily a backup mechanism. All you need to know is how to initialize a new `git` repository, add and commit files to it, and push it to the remote.

## Viewing `git` logs and `diff`s
At the next stage, start using the commits in a meaningful way. Get a list of commit messages by typing `git log`. For me, this is a useful reminder about where I had left off on a project that I may not have worked on for a while.

As you work on your code, the ability to see the differences between the last commit and your current uncommitted changes with `git diff` will be invaluable for debugging.

You can also use `git show <commit-hash>` to see the changes made in any given commit. I also occasionally use the tool called `gitk`, primarily to step through the changes made at each commit. This is done using `gitk <filepath>`.

## Undoing changes
Sometimes I'd mess up a file badly enough to the point where it seemed better to abandon the changes than to continue debugging. In such a situation, as long as you have not yet committed the file, it's easy enough to undo all the changes in the file with:
```bash
git checkout -- filepath
```

This approach is more reliable than depending on your code editor's undo capability. Interestingly, the more sophisticated I became with `git`, the occurrence of such bad messes became fewer and far between.

For undoing a commit, there are a plethora of ways depending on how you want to manipulate `git` history (which you shouldn't worry about at this stage). A simple and effective approach for undoing a commit is:
```bash
git revert <commit-hash>
```
With the above method, you don't have to be concerned about needing to perform forced updates to the remote.

## Using branches to add features
Always use a new branch to add a feature to a working code base. Be disciplined about this. You'd typically keep your current working copy of the code in the `master` branch. And then create a branch off of `master` to add a new feature:
```bash
git checkout master
git pull
git checkout -b feature-xyz
```

Once the new feature has been tested, then you'd merge the feature branch back into `master`, with steps along the following lines:
```bash
# assuming feature-xyz is complete and pushed
git checkout master
git pull
git merge feature-xyz
git push
```

In a one-person project, you'd never have merge conflicts, so this will always work nicely even when using different computers to access your repository. It's usually a good idea to keep the feature branch around for a short while, but it should eventually be deleted locally and on the remote:
```bash
git checkout master
git push origin --delete feature-xyz  # delete feature branch on remote
git branch -d feature-xyz             # delete feature branch locally
```

## Undoing changes with the help of branches
Once you start using branches, more sophisticated ways of undoing changes become available. When things are completely out of control and lots of changes have already been committed, you can always just abandon the branch:
```bash
# Force branch to be deleted without merging to master
git branch -D branch-being-abandoned

# Optional: delete remote branch
git push origin --delete branch-being-abandoned
```

And start back again with a new feature branch off `master`. Sometimes, it's good to save the branch being abandoned for a little while as you might be able to salvage a file or two and add it to your new feature branch. This can be done with:
```bash
git checkout master
git pull
git checkout -b new-feature-branch
git checkout branch-being-abandoned -- good-file
```

The above retrieves the file `good-file` from the `branch-being-abandoned` into your new feature branch.

## Even more sophisticated ways of undoing changes
Then there is `git reset` to manipulate the `HEAD` pointer which you can explore further until your head hurts. But here's a basic use case that will come in handy. Let's say you've made changes to multiple files, performed a commit and pushed it to the remote. And now you're sorry you did all that and want to abandon this last commit. Just do this:
```bash
git reset --hard HEAD~1  # get rid of the last commit completely!
git push -f              # leave no evidence of the last commit in the remote!
```

## Comparing files between branches
If you need to compare `master` and `feature-xyz` branches:
```bash
# Show all changes between these 2 branches
git diff master..feature-xyz
```

The above will show the differences in every file between these branches. And the output can be voluminous if there are lots of files with changes. I often just want a list of files that have changed, which can be obtained with:
```bash
# List files that are different between these 2 branches
git diff master..feature-xyz --stat
```

After that, I can view changes in one particular file between the branches with:
```bash
git diff master..feature-xyz -- <filepath>
```

## Graphical `git` tools vs the command line
There are many great graphical tools and editor extensions that help with using `git` in various ways. I mentioned `gitk` earlier. Tools try to abstract away the complexity of `git` while introducing ambiguity about what they are actually doing under the hood. My preference is to do almost all `git` related actions on the command line. Otherwise, I think it's difficult to truly understand some of the fundamentals.

I use graphical `git` tools to support visualization. A couple of key examples:

- Many editors can be very helpful with `git` merges

- Tools such as `gitk` are great for walking through changes in a file commit-by-commit

## Conclusion
I hope this article helps those at the early stages of their `git` journey with some useful hints. In the next article, I'll talk about more advanced `git` topics.
