# Git

## Foreword

As a developer, Git is probably _the most_ indispensable tool at your disposal. There are only so many iterations of `main_finalfinalfinal.c` you can go through before you end up flirting with the edge of madness. Learning Git is an absolute MUST, and so if you haven't already, I cannot recommend it enough. As always, we'll only be covering some quick tips in this entry. The rest is left as an exercise for the reader. Let's get straight to business

## Writing good commit messages

We've all seen the hideous "Files added via upload". Trust me, no one wants to see that, and if they do, they're very likely not to take your work seriously. This just screams "I don't know Git, but I wrote everything in one go and uploaded the entire shebang wholesale to {insert version control platform domain dot com here}"

When writing commit messages, you're not only highlighting the changes to code for yourself to follow where you left off at any point later, but also to your "users"; i.e. other users (developers or not) using and/or contributing to your project

Commit messages should be descriptive of the changes you introduced. Let's assume the following changes:

```diff
diff --git a/logger.c b/logger.c
index a25714af415b..90025508aadc 100644
--- a/logger.c
+++ b/logger.c
@@ -2,6 +2,7 @@
 #include <sys/stat.h>
 #include <fcntl.h>
 #include <unistd.h>
+#include <errno.h>

 int
 main (void)
@@ -9,8 +10,8 @@ main (void)
        int fd;

        if ((fd = open("/nonexistent", O_RDWR)) == -1) {
-               printf("[-] Operation failed. File either does not exist, "
-                       "or there is an issue with access permissions. "
-                       "Try again later\n");
-               return 1;
+               perror("[-] Could not open file.");
+               return errno;
        }

        /*
```

The above example is really just for demonstration purposes, but let's see two possible ways we can go about communicating those changes to users. One could very well commit something like `YOLO`, or `lol`, but this is not going to help later on. It gets even worse if your project will be open to contributions, or if you have to table it for a good few months only to get back with an "I have no memory of this place" look on your face

Ideally, as previously mentioned, commit messages should be descriptive, but also straight to the point. That is, you do not need to write down the declaration of independence; only what is absolutely necessary to communicate. A better commit message would be somewhat as follows:

```
Clarify error message on failure to open file

* Use `perror` to output the error message to STDERR 
  instead of `printf` to STDOUT
* Return error message resulting from the failed call to open
```

For something as simple as the above change, we can definitely do away with the meticulous explanation since anyone familiar with the language can easily tell. However, there is some value to leaving _some_ sort of explanation especially when first starting out, or if the goal is to also educate beginners looking at your code. We could simply leave it at:

```
Clarify error message on failure to open file and return errno
```

For inspiration on how to write proper commit messages, pull the Linux kernel and start reading through the commit log. Do note, however, that kernel developers, maintainers, reviewers, and contributors have to abide by a very strict set of rules that apply to just about every step of the process. You don't have to follow their exact footsteps, for non kernel-related work, as that might not necessarily be applicable to every other project, but it gives you a great reason to start doing things correctly

## Oh, no, I made a typo in my commit message

First off, there are multiple scenarios that can be the case here:

- **Scenario A:** you've added files for tracking, i.e. `git add <file(s)>`, followed by `git commit -m <message>`, but you didn't `git push`
- **Scenario B:** you've made multiple commits before pushing, but it's not the most recent one that has the typo
- **Scenario C:** you've already pushed to a remote branch

### Scenario A

This one is quite straight forward; `git commit --amend` lets you reword the latest commit before pushing

### Scenario B

Unfortunately, `git commit --amend` won't work for this scenario since the commit in question is not the most recent one. Fret not, however, for Git has got you covered. Let's consider the following example:

```
$ git log --oneline
767edc6fa82e (HEAD -> main) Add test case 3
a46c703c7dd5 Add tst case 23
3479fad7f476 Add test case 1
193eb530337c Initial commit
```

As we can see, we messed up commit `a46c703c7dd5`, and now we need to fix it. Let's start by rebasing that one commit `a46c703c7dd5` as follows:

```
# HEAD is where we're currently at
# We need the two commits including 
# HEAD, so we'll pass HEAD~2
#
$ git rebase -i HEAD~2
```

At this point, your $EDITOR will be forked, with the default action to `pick` (or use) the commit(s). We need to `reword` the commit in question, so we'll change *p*ick to *r*eword (or simply `r`). For other commits, we can either pick or drop them; this is how you can quickly manage your commits in one go. Let's get on with the rewording:

```
r a46c703c7dd5 Add tst case 23
pick 767edc6fa82e Add test case 3

# Rebase 3479fad7f476..767edc6fa82e onto 3479fad7f476 (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

You should see the above displayed in your $EDITOR. Only difference is I've already changed `pick` to `r` (i.e. `reword`). Once you write the file and quit, you'll be forked off to the $EDITOR once more so you can reword the commit(s) in question. Let's fix the embarrassing typos:

```
Add test case 2

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Feb 19 18:41:55 2025 +0200
#
# interactive rebase in progress; onto 3479fad7f476
# Last command done (1 command done):
#    reword a46c703c7dd5 Add tst case 23
# Next command to do (1 remaining command):
#    pick 767edc6fa82e Add test case 3
# You are currently editing a commit while rebasing branch 'main' on '3479fad7f476'.
#
# Changes to be committed:
#       new file:   test2
#
```

A little time-travel doesn't hurt, now does it? If you peek through the output, you'll notice `new file:    test2`. It's as if we've _just_ added that file, but we get a second chance to reword its commit message, without interfering with the later commit(s). Write, quit, and check the log. Et voila:

```
$ git log --oneline
f1641c86eaf9 (HEAD -> main) Add test case 3
27f4c7ea2cff Add test case 2
3479fad7f476 Add test case 1
193eb530337c Initial commit
```

Do note that we now have updated refs for the selected commits, as shown by the change in their SHA-1 IDs. We also saved ourselves the embarrassment of publishing that character soup of a commit message. _Nice!_

One more thing, you absolutely MUST read the comments you get whilst interactively rebasing; they teach about about the different commands you can use, how to abort the ongoing rebase, and what happens when you delete lines. Seriously, read it!

### Scenario C

You done messed up, A-A-ron! Let's fix it

<div class="warning">
<strong>Proceed with caution!</strong>

This involves force-pushing to remote repositories, which means potentially destructive changes if due caution is not exercised
</div>

```sh
$ git log --oneline -1
fa93c7289e84 (HEAD -> main, origin/main) lol
$ git rebase -i HEAD~1
```

$EDITOR forks

```
r fa93c7289e84 lol

# Rebase 22241e9fde5b..fa93c7289e84 onto 22241e9fde5b (1 command)
# ...
```

$EDITOR, again, for rewording

```
Add test file

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Feb 21 02:27:45 2025 +0200
#
# interactive rebase in progress; onto 22241e9fde5b
# Last command done (1 command done):
#    reword fa93c7289e84 lol
# No commands remaining.
# You are currently editing a commit while rebasing branch 'main' on '22241e9fde5b'.
#
# Changes to be committed:
#       new file:   test
#
```

```
$ git log --oneline -1
1076b7c647aa (HEAD -> main) Add test file
$ git push --force -u origin main
...
$ git log --oneline -1
1076b7c647aa (HEAD -> main, origin/main) Add test file
```

Navigating to the remote repository, we'll find that we've reworded the commit in-place, as opposed to creating a new commit on top of the mistaken one

<img class="center" alt="Screenshot showing commit reworded with forced push to remote repository" src="../../assets/img/screenshots/tips/git-rebase-force.png"/>

## Closing thoughts

Learning Git will make your life so much easier. If you're beginning to learn Git, create a private repository and practice all you can. To quickly pull up the man-pages for whichever git command, you can execute `man git push`, or `man git-push`, or `git push --help`

Git responsibly ;)
