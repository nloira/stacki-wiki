## Stacki Development

This document is primarily for core Stacki developers, but it also
applies to anyone contributing to Stacki, with the exception that
external contributions should all be pull requests.

### Coding Standards

#### Whitespace

A lot of our code is Python3, which means everyone needs to agree on
the tabs vs. spaces nonsense. We chose tabs, as in actual tab
characters for all indentation. If your favorite editor inserts
spaces when you hit tab, fix your editor.

#### IDE

ID-what? Use what ever you want, just follow the above rule.

#### Style

We need a real consensus on this, but right now the rule is when you
drop into a piece of code try to copy the existing style. For new code
just keep it readable. Eventually someone will get bored and write up
a style guide for Stacki, we will fight about it for a week or so and
loose interest and we will be back to where we are now. All of this
has happened before, and all of this will happen again.

There is a `.flake8` (think `pep8` with more stuff) in the Stacki
repository that turns off a lot of warnings related to whitespace. You
should run `flake8` often and fix reported issues when possible.

### Source Control Model

All of our Stacki code resides in Git, all of it, if it isn't in Git
it does not exist. If you have code that isn't in Git shame on you, if
you still work here shame on us. Keep your code in Git.

If you struggle with Git ask for help. Git isn't rocket surgery, but
it is at times magic.

Open-source is all on github.com and closed-source is all on our
private GitHub Enterprise, if you have doubts about something being
open-source put it in GHE first and then we can figure out what should
be open (Hint: usually everything). But just keep in mind, once it
goes to github.com the cat is out of the bag.

We follow the
[git-flow](http://nvie.com/posts/a-successful-git-branching-model)
branching model. This doesn't mean everyone actually uses `git-flow`,
but everyone follows the [model](https://danielkummer.github.io/git-flow-cheatsheet).
Which means the repository always has both a `master` and `develop`
branch. So to checkout the code you will always have to checkout both
branches. The idea is development happens in `develop` and only
released code ever makes it to `master`.

```
$ git clone git@github.com:Teradata/stacki
$ cd stacki
$ git checkout -b develop origin/develop
```

You are also encouraged to use `git-flow-avh` which requires you to
initialize the repository after a fresh clone. But, before you even do
that you may need to
[install](https://github.com/petervanderdoes/gitflow-avh/wiki/Installation)
`git-flow-avh`. If you are already fimiliar with `git-flow` you know
the project is dead, `git-flow-avh` has taken over the project and
added thing like `bugfix` branches (analog to `feature` branches).


```
$ git-flow init

Which branch should be used for bringing forth production releases?
   - develop
   - master
Branch name for production releases: [master] 

Which branch should be used for integration of the "next release"?
   - develop
Branch name for "next release" development: [develop] 

How to name your supporting branch prefixes?
Feature branches? [feature/] 
Bugfix branches? [bugfix/] 
Release branches? [release/] 
Hotfix branches? [hotfix/] 
Support branches? [support/] 
Version tag prefix? [] 
Hooks and filters directory? [~/stacki/.git/hooks] 
```


#### Feature Branches

All development should be done on `feature` branches. Once a feature is
ready to be shared with the team and has passed the nightly build and
test system it can be added to the list of branches to be merged back
onto `develop` during the team's weekly merge party. Exceptions will
happen but the general rule here is always stay on a feature branch,
and merge back only as a group.

To start a feature branch:

```
$ git flow feature start name-of-feature
```

This will create a new branch (and check it out) based on `develop`
called `feature/name-of-feature`

While you are developing on your feature branch remember to merge
`develop` back onto your branch often. Failure to merge often will
result in the amusement of your co-workers while you struggle with a
10,000 line merge conflict.

It is up to you if you want to rebase or merge `develop` onto your
branch, but be aware that rebasing means you cannot share your feature
branch with anyone.


#### Bugfix Branches

A `bugfix/` branch is exactly the same as `feature` branches but is
for bugfixes.

To start a bugfix branch:

```
$ git flow bugfix start name-of-bugfix
```

This will create a new branch (and check it out) based on `develop`
called `bugfix/name-of-bugfix`


#### Code Review

The next step to getting your feature back into the `develop` branch
is an optional code review, not everything will go through this but it
is encouraged and sometimes required. Features will usually get
identified as code review candidates during the Merge Party.

If your feature branch requires a code review the group will identify
the person to review and they are then responsible for going through
the code and giving the all clear before the code makes it to the next
Merge Party.

If you think your code is more in the *works for me* state, you need a
code review. It happens to all of us, own up to it and ask for
help. If you don't know if your code needs review, it needs review.

#### Merge Party Prep

Your code needs to be revertible, which means a single commit ready to
merge back into develop.  Additionally the single commit must match
the [commit message format below](#commit-message-format).  Although not
currently required, it is *highly suggested* that your branch also contain
tests which exercise your code in a variety of ways.  Tools to
facilitate/enforce (depending on your point of view ;) these requirements
are on the horizon.

Unless you know better, you should almost certainly be based off of
develop.  Periodically, (and definitely just before the Merge Party)
you should rebase your work on top of develop.  To do so:

```
# git fetch origin develop
# git checkout my_branch
# git rebase develop
```

This will grab the latest changes from develop and put them on your 
branch, then automatically "replay" your commits.  To verify your 
branch has all the commits:

```
git rev-list --count my_local_branch..origin/develop
0
# ^--- number of commits develop has that your branch does not.
# You want this number to be exactly 0.
```

To see if your branch has too many commits:

```
# git rev-list --count origin/develop..my_branch
1
# ^--- the number of commits your branch has that develop does not.
# You want this number to be exactly 1.
```

To reduce your `feature` or `bugfix` changes to a single commit you
will need to rebase you branch.

```
$ # ensure develop is current
$ git fetch origin develop
$ git checkout feature/name-of-feature
$ git rebase -i develop
```

This will re-write your commit history and allow your to interactively
use git's rebase feature to collapse all your commits into a single
commit and update the log message to the correct format. You will need
to `pick` the first commit in the list and change all the others to
`squash`. Just like a merge a rebase may have conflict, you will need
to manually resolve these and then type `git rebase --continue` to
complete the operation. This may happen several times during the
operation.

Once you have rebased and are ready to merge you need to publish your
changes using `--force`, this sounds dangerous but if the branch is
ready to be merge it is going to be deleted right after the merge
anyway.

```
$ git push --force origin feature/name-of-feature
```

Finally, verify that your local branch and the branch on GitHub are
identical.

```
$ git diff --stat origin/$branch_name $branch_name
$ 
```

#### Commit Message Format

In order to automate the building of *release docs* that has *New
Features* and *Bug Fix* sections, we will now require the following
format in your commit log prior to merging your code onto the develop
branch.

Bug fixes.

Minimally, the commit log for a bug fix should look like:

```
BUGFIX: One line summary of the bug fix
```

If you’d like to provide more details, then supply one line summary, then a blank line, then
your details:

```
BUGFIX: One line summary of the bug fix

More info about the bug fix.
And remember, this data is going to go into a document that is readable by humans.
and even more data about the bug fix
```

If you’d like to refer to a Jira ticket in your summary, make sure you have *JIRA* somewhere
in the body:

```
BUGFIX: one line summary of the bug fix

more info about the bug fix
and even more data about the bug fix

This fixes JIRA: STACKI-###
```

If your commit will _break_ existing Stacki installations (e.g., a schema change), make sure
to put *BREAKING CHANGE* in the body of the commit message.

```
BUGFIX: one line summary of the bug fix

BREAKING CHANGE: 
This commit breaks Stacki, and here is an explanation on how you can update your frontend
so you can apply this commit and still have a functioning system.
```

All text after *BREAKING CHANGE* will not be included in the release docs.

If you have text that you’d like to include in the commit message, but you don’t want the
text to be in the release docs, then use *INTERNAL*:

```
BUGFIX: one line summary of the bug fix

INTERNAL:
All text from the line above and to the end of the commit message will not be included
in the release docs.
```

You can also use *INTERNAL* in the one line summary:

```
INTERNAL: This commit log message should not be in the release docs.
```

If your commit is a new feature, then your commit logs will look like:

```
FEATURE: one line summary of the feature
```

or

```
FEATURE: one line summary of the feature

more info about the commit

INTERNAL:
all the content after this is ignored and will not be put in the document
```

If your commit is related to documentation, use *DOCS*:

```
DOCS: A documentation commit

More info goes here
```

#### Read Only Friday

To qualify for merging on Monday, *your branch must be completely ready
on Friday*.  This means clean commit history, passed automated and 
manual testing, and code reviews complete by Friday morning.  The Branch
Manager will take Friday to assemble a staging branch with all of the
"Ready" branches, and run that through CI.

#### Monday Merge Party

For the merge party on Monday every developer with a completed feature
will sit in a room together to discuss and merge their branches back
onto `develop`, and immediately get `develop` back into the build and
test system.  Not all features invited to the party get merged, some get
pushed back to code review, and some may get deferred for a later party.

The Branch Manager will run the meeting, which involves going down the
list of merge-ready branches and for each one, triple-checking the diff,
the commit log, and (hopefully in the future) the test status.  For the
sake of Sanity, start from a clean clone of Stacki.

The Branch Manager will then ask the developer for a final confirmation
before performing the merge onto the `develop` branch.

To prevent "merge commits", the Branch Manager should perform a rebase
on develop for each branch just before merging.  There is no need to
push these rebased branches out, as we will be deleting them once
merged.

```
# git rebase develop $next_branch_name
# git checkout develop
# git merge $next_branch_name
# GOTO 10
# Maybe there's a "git flow" way to do the above?
```

This will result in fast-forward merges except in the cases of actual
merge conflicts, which obviously have to be resolved normally.


### Release Branches

Releases begin after the desired feature branches are all merged onto
the `develop` branch and it has been tested in the nightly build and
test system. The release process begins with a fresh clone on the
Stacki repository, and since we are releasing from the `develop`
branch that needs to be checked out as well. We know, it's perfectly
safe to start from an existing repository, but don't.

```
$ git clone git@github.com:Teradata/stacki.git
$ cd stacki
$ git checkout -b develop origin/develop
```

Unlike the development process, during we release we strictly require
the use of `git-flow` to start a new release branch and eventually
merge everything back onto `develop` and `master`. *Note* that we
accept the default values for all of `git-flow init` except for the
`Version tag prefix` which should be set to `stacki-` (yes you need
the dash).

```
$ git-flow init

Which branch should be used for bringing forth production releases?
   - develop
   - master
Branch name for production releases: [master]

Which branch should be used for integration of the "next release"?
   - develop
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? [] stacki-
Hooks and filters directory? [~/stacki/.git/hooks]
```

Next we decide on the name for the release (e.g. 5.1rc1) and start the
release. This will create a new local branch called `release/5.1rc1`
based on `develop` and checkout the branch.

```
$ git flow release start 5.1rc1
```

This is when the version number is changed, in `stacki/version.mk`
change the `ROLLVERSION` variable to the release name (e.g. 5.1rc1)
and checkin the change with a comment indicating the release is started.

```
$ vi version.mk
$ git add version.mk
$ git commit -m "starting release 5.1rc1"
```

Once the release is started only bug fixes are allowed to be checked
into the release branch. Development on feature branches and merge
parties continue, and have no impact on the release branch. Bug fixes
may be direct commits to the release branch or cherry-picked commits
from other branches.

But first push the branch back to the origin so everyone sees it.

```
$ git flow release publish 5.1rc1
```

Once all bugs are squashed and the code is ready to ship the
branch is merge back onto both `develop` and `master`, this ensures
any bugfixes applied only to the release branch go back into `develop`
and that `master` only contains released code. Finishing the release
will also create a an annotated tag based on the release name (in this
example the tag will be `stacki-5.1rc1`) which means it will ask for a
comment (same as doing a `git commit`). This message is stored and can
be accessed with `git show stacki-5.1rc1`).

The following will finish the release, or as we say *tag it, and bag it*.

```
$ git flow release finish 5.1rc1
$ git push
$ git push --tags
```

#### Release Candidates vs Releases

The above procedure walked through the process of building a *release
candidate*, not a full blown release to open-source. Often times it
will take several iterations of candidates to settle on a formal
release. At this time the current candidate is promote to
release. There are no code changes from the final release candidate to
release. The only change is to clean up the version number so it
doesn't say "rc", freak people out and generate email (or worse a Jira
ticket).

To do this we modify the `master` branch since that is where the
current candidate code is, and `develop` already has new pre-release
code in it.  Assume that `stacki-5.1rc4` was the final candidate and
we want to promote it to release. This is done without `git-flow`.

```
$ git checkout master
$ git diff stacki-5.1rc4
```

If the `git diff` above showed that `master` is not the same as
`stacki-5.1rc4` badness happened. Go figure out who messed up the
`master` branch and start reverting (the commits and the
developer). If everyone was good (or you fixed it), continue with the
following:

```
$ vi version.mk
$ git commit -a -m "stacki-5.1 release"
$ git tag stacki-5.1
$ git push
$ git push --tags
$ git checkout develop
$ git merge master
$ git push
```

Now re-build it and release it into the wilderness.










