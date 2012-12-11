# A Time Traveler's Guide to Git

or

# 5 Ways to Rewrite History with Git

## Introduction

While scientists have crushed the dream of traveling back in time, [Git](http://git-scm.com/) offers control over the fourth dimension for times when the wrongs of the past need corrected. The distributed version control system allows commits to be amended, discarded, reordered and modified to scrub data from a repository.

Before the excitement of witnessing this rare phenomena becomes overwhelming, heed the warnings of an experienced time traveler. Git obeys the law of causality; every commit in a git repository is inextricably linked to the commit before it. Changing one commit alters all the commits that come after, creating an alternate reality. Altering the past can be dangerous and–except in rare circumstances–should only be done if the events being altered have not been observed by anyone else. Branches that have already been pushed to a remote should generally never be altered.

Hold on as we explore ways to rewrite history with git.

>> This section sounds like github :heart: but it seems like it could be even better.
>> For example, instead of "Hold on as we explore...", maybe "Join me as we explore..."?
>> Or "... should generally never ..." could be "... should not ..."

## Amend Recent History

The safest and likely most common form of rewriting the git history is to amend the latest commit. For whatever reason, the human brain seems wired to remember something important just after pressing the "Send" button on an email, and the right words always come to mind after a conversation is over. Likewise, I often realize I made a mistake immediately after making a commit in git.

This article was written in a [git repository](https://github.com/bkeepers/git-history). The first commit was to create a README explaining the purpose of the repository.

	$ git add README.md
	$ git commit -am 'Add README'
	[master (root-commit) 6261ead] Add README
	 2 files changed, 12 insertions(+)
	 create mode 100644 README.md
	 create mode 100644 article.md

Oops, after committing, I realized that I had committed `article.md`, which was just some notes and the first few sentences of the introduction. I did not intend to commit that file yet, so let's remove it from the history.

	$ git rm --cached article.md
	rm 'article.md'

The `--cached` argument to `git rm` tells git to stage the removal of the file, but to not actually delete the file from the filesystem.

We can also make other modifications like we would if we were going to create another commit–such as making edits to the README.md and staging them with `git add`. Amend the previous commit by passing the `--amend` flag to `git commit`:

	$ git commit --amend
	[master 667f8c9] Add README
	 1 file changed, 7 insertions(+)
	 create mode 100644 README.md

Git will open an editor to allow editing the previous commit message. The git log now shows that there is still only one commit, and that commit only has `README.md`.

	$ git log --oneline --stat
	667f8c9 Add README
	 README.md | 7 +++++++
	 1 file changed, 7 insertions(+)

Let's commit this article now that progress has been made.

	$ git add article.md
	$ git commit -m 'first draft of amend section'
	[master 8dbf5d5] first draft of amend section
	1 file changed, 47 insertions(+)
	create mode 100644 article.md

## Undo Recent History

Sometimes, a commit has so many things wrong that that it is easier to just undo it. Maybe it was committed to the wrong branch, or a directory of unwanted files got accidentally added.

	$ git reset HEAD^

This tells git to reset to the previous commit, but to keep the changes introduced by that commit. Looking at the log and status, we can see that our latest commit is gone, but `article.md` is still modified.

>> Maybe add an abbreviated version of [reset demystified](http://git-scm.com/2011/07/11/reset.html), or at least a reference to it?

	$ git log --oneline
	667f8c9 Add README
	
	$ git status -s
	M  article.md

From here, the changes can be committed on a different branch, stashed, discarded or modified and recommitted.

## Maintain a Tidy History

If you have used git with a team, then there is no doubt that you have seen a push get rejected.

	$ git push origin master
	To git@github.com:bkeepers/git-history.git
	 ! [rejected]        master -> master (non-fast-forward)
	error: failed to push some refs to 'git@github.com:bkeepers/git-history.git'
	hint: Updates were rejected because the tip of your current branch is behind
	hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
	hint: before pushing again.
	hint: See the 'Note about fast-forwards' in 'git push --help' for details.

While this message looks big and scary, it is actually quite helpful. The hints tell us that since we started our work, one of our team members pushed changes and we need to get them, usually by running `git pull`. The hint also recommends checking out the [note about "fast-forwards"](http://www.kernel.org/pub/software/scm/git/docs/git-push.html#_note_about_fast_forwards) in the git docs. I second that recommendation.

Running `git pull` will fetch the remote changes and create a new commit that merges them with our local changes. While there is nothing wrong with the merge commit, it adds unnecessary complexity to the revision history.

	$ git log --decorate --graph --oneline
	*   6341ecc (HEAD, master) Merge branch 'master' of https://github.com/bkeepers/git-history into master
	|\
	| * f82686e (origin/master, origin/HEAD) edits
	* | d60bf73 Rebase example.
	|/
	* 667f8c9 Add README

What would be more ideal would be to take our changes and apply them on top of the remote changes.

	$ git pull --rebase origin master
	First, rewinding head to replay your work on top of it...
	Applying: update README

This makes the revision history appear as if the change was made after a teammate made their commit.

	$ git log --oneline
	* 4defdea (HEAD, master) Rebase example.
	* f82686e (origin/master, origin/HEAD) edits
	* 667f8c9 Add README

Unless a repository is being pushed to multiple remotes, rebasing when pulling is almost always a good idea. I have git configured to configured to rebase automatically.

    $ git config --global branch.autosetuprebase always

Keeping the revision history tidy may seem superficial, but it helps immensely when managing a large project.

## Cleanup Recent History

Sometimes it is not clear until a few steps later that you are on the wrong path. Git's flexibility makes it easy to create checkpoints along the way, offering a point to return to if things go wrong.

In my daily development, I commit as often as possible. Anytime I think to myself "ok, that is done, now what?", I commit. While this leads to a revision history that accurately reflects the order of events, it is not the most conducive to effectively managing a large project. So once I am ready to share my changes with my team, I review my unpublished commits and clean them up.

An interactive rebase allows commits to be edited, squashed together or completely removed from the recent history of a branch.

While reviewing my progress on this article, I discovered a few embarrassing typos. Since the repository had not been shared with anyone yet, I covered my tracks by fixing the typos in the original commit. I preserved my original mistake, so you can follow along by checking out the [typos](https://github.com/bkeepers/git-history/tree/typos) branch of the repository.

First, I created two new commits to fix the typos.

	$ git log --oneline
	7445019 Fix misspelling of amend
	b0377f9 Fix typo in title
	b1cdd72 first draft of pull --rebase
	2fbe35b first draft of reset
	7bb9109 first draft of amend section
	667f8c9 Add README

Take note of the commit that needs to be fixed up. Both of the typos were from commit `7bb9109`, "first draft of amend section". Start the rebase at the revision before:

	$ git rebase -i 7bb9109^

Git will open the editor with the list of commits and a very helpful message.

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

As the note explains, commits can be rearrange to change their order, or `pick` can be changed to one of the other commands.

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

The log shows that the typo fixing commits are now gone. The fixes were applied to the original commits and there is no evidence of my poor spelling (in this branch).

	$ git log --oneline
	4787614 first draft of pull --rebase
	ee719e9 first draft of reset
	00165a8 first draft of amend section
	667f8c9 Add README

This rebase worked without any other interaction, but occasionally a rebase will require manual fixes for merge conflicts. If that happens, don't freak out. Simply read the messages. Git will usually help get you out of a bind.

## Rewrite All of History

All the git commands we have examined so far are useful for modifying recent commits, but sometimes more extreme measures are necessary, whether it is to remove sensitive or extremely large files or to simply make a project easier to manager.

[`git filter-branch`](http://git-scm.com/docs/git-filter-branch) supports a hand full of custom filters that can rewrite the revision history for a range of commits.

My first legitimate use of `git filter-branch` was on large project where the server and the client were both in the same repository. As more people were added to the team and tensions between the hipsters and neck-beards rose, it became obvious that two repositories would be more appropriate. A simple solution would have been to clone the repository twice, delete the unneeded files, and move the remaining files around. But that leaves two repositories with duplicate histories that take up unnecessary space. Instead, we cloned the repository twice, and used the `--subdirectory-filter` to create two new repositories that only contained the changes for the relevant parts of the application.

	$ git filter-branch --subdirectory-filter client -- --all

Many people use different emails for personal and work projects, which can easily result in commits to a repository using the wrong email address. The `--env-filter` can modify basic metadata about a commit, such as author information or the commit date.

	$ git filter-branch --env-filter '
	  if [ $GIT_AUTHOR_EMAIL = personal@example.com ];
	  then GIT_AUTHOR_EMAIL=work@example.com;
	  fi; export GIT_AUTHOR_EMAIL'
	Rewrite f853027b7979756bab7146d3bb34d8829b81a884 (8/8)
	Ref 'refs/heads/master' was rewritten

Maybe early on in a project someone committed some extremely large assets, and now everyone that clones the repository has to wait for those assets to download.  Or you're open-sourcing a project that has some sensitive data (e.g. passwords) stored in it.

	$ git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch docs/designs' \
	  --prune-empty --tag-name-filter cat -- --all

All of the following changes will rewrite the full history of a repository, essentially making it a new repository. Pushing to the same remote that was used originally will get rejected.

    $ git push
	 ! [rejected]        master -> master (non-fast-forward)

It is possible to force git to push all changes to an existing remote, but remember that this could have adverse effects for everyone else working on the project.

    $ git push --force --all --tags

## Conclusion

Git's powerful features, extreme flexibility and often unintuitive command line may seem overwhelming, but taking time to learn and experiment is a worth-while investment. Understanding how and when to rewrite the revision history will give you complete control over your projects and make them easier to manage.
