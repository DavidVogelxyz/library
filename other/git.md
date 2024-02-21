# Git

## Table of contents

- [The correct way to do a rollback](#The-correct-way-to-do-a-rollback)
- [Patch adding and patch committing](#Patch-adding-and-patch-committing)

## The correct way to do a rollback

There have been times when I've made commits in a less-than-preferred manner.

An example of this is when I make a commit, and immediately notice a small typo. I find it irritating that the typo wasn't addressed in the original commit, and I'm reluctant to to push an additional commit just to correct a minor typo.

In these instances, I would vastly prefer to rollback to the previous "good" commit, and then push one correct commit.

I have found the best way to address this is the following.

```
git reset --hard $hash_of_desired_commit
```

The above command will rollback the repo to the state it was in before I pushed the commit with the minor error. It also reverts the HEAD back to that commit. Then, I perform the following command:

```
git clean -df
```

The above command cleans up the repo so that everything is exactly the way it was before the commit took place. I then make the changes I wanted to be included in the "one good commit."

Finally, I commit the changes and then push them using the following command:

```
git push -fu origin master
```

Normally, I use `git push -u origin master`. However, in this case, I require a --force flag to overwrite the commit that was already pushed to the public repo. While I could write the flags as `-uf`, I find the other way to be more amusing.

## Patch adding and patch committing

Sometimes, I don't want to add or commit all the changes in a file. Rather, I only want to commit *some* of the changes made to a file.

There's a really great flag for this: "-p"

The "p" stands for "patch," and it's the solution to this problem.

An example use case is making some changes to a vimrc file. There have been times when I have some changes to that vimrc file that I don't want to commit. However, one day, I make a small change that I *do* want to commit. Instead of trying to fix the vimrc file so that the only changes are the ones I want to commit, I can use a patch commit instead:

```
git commit -pm "commit message"
```

NB: this should be obvious, but if the example vimrc file was staged using `git add`, then the "-p" won't work as intended. This is because all the changes to that file have already been staged. In order for the "-p" to work as intended, the file has to be a tracked file without being a staged file.

By running `git commit -p`, git will look at all staged files that have changes, and will go through chunks of the file to ask which chunks should be staged. This gives me some discretion about what changes should be included in the commit, and which should not.
