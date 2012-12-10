# A Time Traveler's Guide to Git

## Introduction

While scientists tell us that traveling back in time is an impossibility, [Git's](http://git-scm.com/) features and flexibility offer control over the fourth dimension for those times when the wrongs of the past need corrected. The distributed version control system allows you to easily amend or undo recent commits, reorder changes and scrub data from your repository.

Before the excitement of witnessing this rare phenomena becomes overwhelming, heed the warnings of an experienced time traveler. Git obeys the law of causality; every commit in a git repository is inextricably linked to the commit before it. Changing one commit alters all the commits that come after. Altering the past can be dangerous and–except in rare circumstances–should only be done if nobody else has observed the events that you are altering. Branches that have already been pushed to a remote should generally never be altered.

Understanding which operations are safe and which are dangerous can lead to a cleaner git history and make projects easier to manage.

## Amend the latest commit

The safest and likely most common form of rewriting the git history is to amend the latest commit. For whatever reason, our brains seem wired to remember something important just after pressing the "Send" button on an email. Likewise, I often realize I made a mistake immediately after making a commit in git.

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

We can also make other modifications and stage them like we would if we were going to create another commit. Amend the previous commit by simply passing the `--amend` flag to `git commit`:

	$ git commit --amend
	[master 667f8c9] Add README
	 1 file changed, 7 insertions(+)
	 create mode 100644 README.md

Git will open an editor to allow editing the commit message. The git log shows that there is still only one commit, and that commit only has `README.md`.

	$ git log --oneline --stat
	667f8c9 Add README
	 README.md | 7 +++++++
	 1 file changed, 7 insertions(+)

Now that We've made some progress, let's commit our progress on this article:

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

	$ git pull --rebase origin master
	First, rewinding head to replay your work on top of it...
	Applying: update README

	$ git log --oneline
	c408281 update README
	d136ca4 first draft of reset
	8dbf5d5 first draft of amend section
	667f8c9 Add README

Rebasing when pulling is almost always a good idea, so you might want to configure git to rebase by default.

    $ git config --global branch.autosetuprebase always

# Interactive Rebase

An interactive rebase allows you to edit commits, squash multiple commits together or completely remove commits from from the recent history of your branch.  It is extremely useful for cleaning up a local branch before pushing to a remote.

While I was reviewing my progress of this article up to this point, I discovered a few embarrassing typos. Since my git repo had not been shared with anyone yet, I covered my tracks by fixing the typos in the original commit. Here is how I did it. I preserved my original mistake, so you can follow along by checking out the [typos](https://github.com/bkeepers/git-history/tree/typos) branch of the repository.

First, I created two new commits that fixed the typos.

	$ git log --oneline
	7445019 Fix misspelling of amend
	b0377f9 Fix typo in title
	b1cdd72 first draft of pull --rebase
	2fbe35b first draft of reset
	7bb9109 first draft of amend section
	667f8c9 Add README

Then I need to take note of the commit that needs fixed up. Both the typos were from commit `7bb9109`, first draft of amend section. So I start the rebase at the revision before:

	$ git rebase -i 7bb9109^

Git will now open the editor with the list of commits and a very helpful message.

	pick 7bb9109 first draft of amend section
	pick 2fbe35b first draft of reset
	pick b1cdd72 first draft of pull --rebase
	pick b0377f9 Fix typo in title
	pick 7445019 Fix misspelling of amend

	# Rebase 667f8c9..7445019 onto 667f8c9
	#
	# Commands:
	#  p, pick = use commit
	#  r, reword = use commit, but edit the commit message
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#  f, fixup = like "squash", but discard this commit's log message
	#  x, exec = run command (the rest of the line) using shell
	#
	# These lines can be re-ordered; they are executed from top to bottom.
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	#
	# However, if you remove everything, the rebase will be aborted.
	#
	# Note that empty commits are commented out

As the note says, you can rearrange the order of the commits, and change `pick` to one of the other commands.

	pick 7bb9109 first draft of amend section
	fixup b0377f9 Fix typo in title
	fixup 7445019 Fix misspelling of amend
	pick 2fbe35b first draft of reset
	pick b1cdd72 first draft of pull --rebase

I moved the two typo fixes to just after the commit where they were introduced and changed `pick` to `fixup` to meld them in to the original commit. After saving and closing the editor, git will apply the changes:

	[detached HEAD 00165a8] first draft of amend section
	 1 file changed, 47 insertions(+)
	 create mode 100644 article.md
	Successfully rebased and updated refs/heads/master.

Now we can see by looking at the log that our history doesn't show the commits fixing the typos.

	$ git log --oneline
	4787614 first draft of pull --rebase
	ee719e9 first draft of reset
	00165a8 first draft of amend section
	667f8c9 Add README

This rebase worked without any other interaction, but occasionally you will have to manually fix merge conflicts. If that happens, read the messages that git gives you and don't freak out. Git will usually help you get out of a bind.

## Rewrite all of history

All of the following changes will rewrite the full history of a repository, essentially making it a new repository. If you try to push it to the same remote that was used originally, it will get rejected.

    $ git push
	 ! [rejected]        master -> master (non-fast-forward)

If you would like to use the same remote, you can force git to push all of your changes, but note that this could have adverse effects for everyone else working on the project.

    $ git push --force --all --tags

[`git filter-branch`](http://git-scm.com/docs/git-filter-branch) supports a hand full of custom filters that can rewrite the revision history for a range of commits.

My first legitimate use of `git filter-branch` was on large project where the server and the client were both in the same repository. As we added more people to the team and tensions between the hipsters and neck-beards rose, it became obvious that two repositories would be more appropriate. We could have simply cloned the repository twice, delete the unneeded files, and moved files around. But that leaves us with two repositories with duplicate histories that take up unnecessary space. Instead, we cloned the repository twice, and used the `--subdirectory-filter` to create two new repositories that only contained the changes for the relevant part of the application.

	$ git filter-branch --subdirectory-filter client -- --all

If you use different emails for personal and work projects, you may have accidentally made commits to a repository using the wrong email address. The `--env-filter` can modify basic metadata about a commit, such as author information or the commit date. 

	$ git filter-branch --env-filter '
	  if [ $GIT_AUTHOR_EMAIL = personal@example.com ];
	  then GIT_AUTHOR_EMAIL=work@example.com;
	  fi; export GIT_AUTHOR_EMAIL'
	Rewrite f853027b7979756bab7146d3bb34d8829b81a884 (8/8)
	Ref 'refs/heads/master' was rewritten

Maybe early on in a project someone committed some extremely large assets, and now everyone that clones the repository has to wait for those assets to download. Or maybe sensitive data found its way into your repository, either on purpose or often by accident.

	$ git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch docs/designs' \
	  --prune-empty --tag-name-filter cat -- --all
