# Config

Setup your commit email address to reflect your GitHub account and set your real name

    > git config --global user.email "you@example.com"
    > git config --global user.name "Full Name"

Set branch merge strategy to rebase

    > git config --global branch.autosetuprebase always

# General

Below is an example for adding a new file

First clone the repository to get a local working copy

    > git clone git@github.com:DNAOY/remote-repo-data

**Add** new file to be committed

    > echo (println "moi") > core.py
    > git add core.py

Now you can check the state of your own working copy with `git status`:

    > git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

	    new file:   core.py

After adding, a commit can be recorded with a message describing what was done

    > git commit -m "Add implementation of new feature"
    [master 3983100] Add implementation of new feature
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 core.py

`git status` will now show that one new commit can be pushed:

    > git status
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
      (use "git push" to publish your local commits)
    nothing to commit, working directory clean

To publish the commit to the central repository, do a **push**

    > git push
    Counting objects: 4, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (4/4), 1.13 MiB | 0 bytes/s, done.
    Total 4 (delta 1), reused 0 (delta 0)
    To git@github.com:DNAOY/remote-repo-data
       ebc8066..3983100  master -> master

To fetch and update the local working copy, do a **pull**

    > git pull --rebase

This will first fetch new commits from the remote and then apply them to local copy. 

`--rebase` will instruct to apply local commits (if any) on top of new commits fetched from the remote (the default it Git is to create a merge commit in this situation, but using rebase avoids this and yields a simpler linear history).

# Feature branches

## Creating a branch

Create a new branch by starting from master

    > git checkout -b feature/components_directory

In the above, `checkout` command will change contents of working directory from one branch to another and `-u` creates a new branch if one does not exist.

Add and commit changes to the newly created branch:

    > git add data_utils.py
    > git commit -m "Add data utils component"

Publish the branch by pushing it to the central repository:

    git push -u origin feature/components_directory

In the above, `origin` tells where to push (by convention, cloning creates a remote named `origin`) and `feature/components_directory` is the name of the branch to create in the remote (prefer same name as locally).

`-u` will mark the local branch as *tracking* the remote branch. This allows `git status` to show if there are commits that haven't been pushed or if there are commits that haven't been pulled from the remote.

If new commits are later on added, they can be published with:

    > git push

## Making a Pull Request

First, ensure your local feature branch is up to date with the remote (in case other collaborators have made changes to the same branch).

    > git checkout feature/components_directory # <- this is the branch from which a Pull request is made from
    > git pull

Next, make sure your local branch is up to date with master. This makes it easier to read changes that are on top of current latest development.

    > git checkout master
    > git pull
    > git checkout feature/components_directory
    > git rebase master
    First, rewinding head to replay your work on top of it...
    Applying: Add calendar component

If `rebase` changes history, you'll have to overwrite history of the feature branch in the central repository:

    > git push --force-with-lease

The "force with lease" option will overwrite the history of the feature branch in the central repository, but will also protect you from accidentally overwriting changes that someone else may have done in the feature branch and which are not included in your local branch history.

After this, start a pull request from Github (for example by navigating to the Pull request page) and ask for comments. After review, merge the branch to master, by first switching to master branch:

    > git checkout master

And then doing the actual merge:

    > git merge --no-ff feature/components_directory

And publish newly updated master branch:

    > git push

Github will then close the Pull request but the branch will stay in the remote. Remove it from origin with:

    > git push origin :feature/components_directory
    To git@github.com:DNAOY/remote-repo-data
     - [deleted]         feature/components_directory

Local branch can be removed with (so that own clone won't fill up with old branches):

    > git branch -d feature/components_directory

# Submodules

A new submodule can be created with:

    git submodule add git@github.com:DNAOY/cip-crawler.git cip-crawler

In the above, `git@github.com:DNAOY/cip-crawler.git` tells the location of the remote repository and `cip-crawler` is the path into which the remote repository is cloned in the current repository (a "supermodule").

`git status` will then tell that the addition of a submodule was recorded and a new path was added:

```
0% git st
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

       	modified:   .gitmodules
       	new file:   cip-crawler
```

Commit the changes with:

```
0% git commit -m "Add cip-crawler submodule"
[feature/cip b5f7d7e] Add cip-crawler submodule
 2 files changed, 4 insertions(+)
 create mode 160000 cip-crawler
```

And publish them with`git push`

After this, other users can do `git submodule update` to make git clone the submodule into their local working copy.

In order to make Jenkins jobs do this, add `Advanced sub-modules behaviours` from which active `Recursively update submodules` option. This will also descend to submodules of the newly added submodule (recursion).

# Subrepo

[git-subrepo](https://github.com/ingydotnet/git-subrepo) is a good alternative to submodules. Main difference is that commit objects of a nested repository live inside another repository. One does not have to to make a commit to a submodule and then take it into use in the repository containing a submodule. Instead, a directory tree is made to contain another repository and changes committed into that tree can be pushed to that repository. Better docs and usage instructions are found in the project site: https://github.com/ingydotnet/git-subrepo

# Aliases

You can define aliases for your commonly used or complicated and hard to remember git commands by using `git config`. For example, you can define an alias `force-push` for the command `push --force-with-lease` like this:

```
git config --global alias.'force-push'='push --force-with-lease'
```