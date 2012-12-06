# A Time Travelor's Guide to Git

## Introduction

Many people will advise you to never modify commits. While they are generally correct, understanding which operations are safe and which are dangerous can lead to a cleaner git history and make projects easier to manage.

## When is it safe to rewrite history?

Branches that have already been pushed to a remote should generally never be rewritten. 

## Amend the latest commit

For whatever reason, our brains seem wired to remember something important just after we hit the "Send" button on an email. Likewise, I often realize I made a mistake immediately after making a commit in git.

The safest and likely most common form of rewriting the git history is to ammend the latest commit.

This article was written in a [git repository](https://github.com/bkeepers/git-history). When I started it, I created a README explaining the purpose of the repository. 

		$ git add README.md
		$ git commit -am 'Add README'
		[master (root-commit) 6261ead] Add README
		 2 files changed, 12 insertions(+)
		 create mode 100644 README.md
		 create mode 100644 article.md

Oops, after I made the commit, I realized that I had committed `article.md`, which was just the first few sentences of the introduction. I did not intend to commit that file yet, so let's remove it.

		$ git rm --cached article.md
		rm 'article.md'

The `--cached` argument to `git rm` tells git to stage the removal of the file, but to not actually delete the file from the filesystem.

We can also make other modifications and stage them like we would if we were going to create another commit.

Amend the previous commit by simply passing the `--amend` flag to `git commit`:

		$ git commit --amend
		[master 667f8c9] Add README
		 1 file changed, 7 insertions(+)
		 create mode 100644 README.md

This should open up an editor to allow editing the commit message. Looking at the git log, we can see that there is still only one commit, and that commit only has `README.md`.

		$ git log --oneline --stat
		667f8c9 Add README
		 README.md | 7 +++++++
		 1 file changed, 7 insertions(+)

We've made some progress, let's commit our progress on this article:

	$ git add article.md
	$ git commit -m 'first draft of amend section'
	[master 8dbf5d5] first draft of amend section
	1 file changed, 47 insertions(+)
	create mode 100644 article.md

## Undo the last commit

Sometimes, a commit has so many things wrong that that it is easier to just undo the whole thing and redo it. Maybe it was committed to the wrong branch, or a bunch of files got added that should not have.

	$ git reset --soft HEAD^

This tells git to reset to the previous commit, but to keep the changes introduced by that commit.

	$ git log --oneline
	667f8c9 Add README
	$ git status -s
	A  article.md

You can see that our commit is now gone, but the changes to `article.md` are still staged.

…

## git pull --rebase

If you have used git with a team, then there is no doubt that you have seen this:

    $ git push origin master
    To git@github.com:bkeepers/git-history.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:bkeepers/git-history.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
    hint: before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.

While this message looks big and scary, it is actually quite helpful. The hints basically tell us that one of our team members have pushed changes and we need to get them, usually by running `git pull`. The hint also recommends checking out the [note about "fast-forwards"](http://www.kernel.org/pub/software/scm/git/docs/git-push.html#_note_about_fast_forwards) in the git docs. I second that recommendation.

Running `git pull` will fetch the remote changes and create a new commit that merges them with our local changes. While there is nothing wrong with the merge commit, it unnecessarily muddies up our history.

[show illustration here]

What would be more ideal would be to take our changes and apply them on top of the remote changes.

``` command-line
	$ git pull --rebase origin master
	> First, rewinding head to replay your work on top of it...
	> Applying: update README
```

	$ git log --oneline
	c408281 update README
	d136ca4 first draft of reset
	8dbf5d5 first draft of amend section
	667f8c9 Add README

Rebasing when pulling is almost always a good idea, so you might want to configure git to rebase by default.
 
    $ git config --global branch.autosetuprebase always

