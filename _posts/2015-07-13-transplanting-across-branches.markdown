---
layout: post
title:  "Transplanting commits across branches"
date:   2015-07-13 09:30:00
author: seth
comments: true
image: /assets/article_images/2015-07-13-transplanting-across-branches/leap_of_faith.jpg
---

In our company, we generally have a pretty simple structure of git branching.

- `master` reflects production.
- `develop` reflects work-in-progress before daily release.
- Anything else is _usually_ a branch off `develop`.

Due to its simplicity we rarely have to deal with anything too complex, just the odd rebase and conflict when
two developers are working on separate branches touching similar code. Any such work is generally solved
with something as simple as.

```
git fetch
git rebase origin/develop
# fix conflicts
git push origin my_branch -f
```

However, in less common circumstances we sometimes have a third layer. Two or more developers may be working
on a longer-term special project that should not go into the daily release. Let's call this project `area52`.
`area52` has its own branches, while other developers are still pushing commits into their branches off `develop`.

Here is what happens if the feature branch `area52` is rebased with conflicts, while other branches are waiting
to be merged into it.

![Conflict creates a new base](/assets/article_images/2015-07-13-transplanting-across-branches/rebase_conflicts.png)

The red circle is the point of our problem. Rebasing the `area52` branch had conflicts which needed
to be fixed, meaning that all of its old commits have been changed! Now, the branch `a52_b` cannot be
merged because it no longer recognises the new commits that it is merging on to. In github you'll see
this with `a52_b`'s pull request suddenly getting huge numbers of commits before it.

We need to take the commits that we have been doing on `a52_b` and apply them on to this new version
of `area52`. We have tried three solutions to this problem:

1. Never rebase a master branch on to a feature branch, and instead only ever use `git merge`. This removes
the above problem but it's not very clean — you have to deal with every conflict in a single commit,
rather than taking each commit at a time.

2. `git cherry-pick` each work-in-progress commit on to the new version of the feature branch. This will
keep the new rebased commit history, but it's a bit tedious especially if you have a lot of commits to
migrate.

3. `git rebase --onto` to rebase across branches.

`--onto` allows us to specify a new base for our rebasing. For the purposes of this problem, it allows us
to take all commits from a set point and drop them anywhere else, including into a different branch,
even between commits.

All we need to do is find the first commit of our branch, in this case `372533c`.

![Get the first commit](/assets/article_images/2015-07-13-transplanting-across-branches/commits.png)

Then, voila!

```
git fetch
git rebase --onto origin/develop 372533c
# fix conflicts
git push origin my_branch -f
```
