---
layout: post
title: "Contributing to open source projects on GitHub - Cheat Sheet"
date: 2014-07-08 01:24:32 +0200
comments: true
categories: GitHub
sidebar: collapse
---

I've been contributing to many open source projects over the past couple of years and I found [GitHub](https://github.com/jaceklaskowski) pleasantly helpful to continue the gig in the years to come. I've learnt few techniques along the way.

I don't want to keep the techniques for myself so the git/GitHub cheat sheet is supposed to help me remember the commands and others to learn from my mistakes (*aka* experience). It's so easy on GitHub that I keep wondering why it took me so long to learn it. It must not for you.

Have fun contributing to open source projects as much as I do! *Pro publico bono.*

**NOTE** It becomes feature-complete when the note disappears. Live with the few mistakes for now. Let me know what you think in the Comments section. Pull requests are welcome, too. Thanks!

<!-- more -->

## Picking project

You start your day hunting down the project you want to contribute to. Be adventurous and pick the one you've always been dreaming about. This might be the day when the dream comes true.

I'm more into Scala/sbt lately so I'm with projects under control of the project build tool - [sbt](http://www.scala-sbt.org/) as I can learn both contributing.

## Cloning project

Learning a project can take different approaches and reading the source code or just building it and staying on the cutting edge are a few examples.

In the project's repository on GitHub, on the right-hand side, there's this clone URL field. Select the protocol to use (HTTPS or SSH) and click the Copy to clipboard button.

In the terminal, execute the following command:

    git clone [clone URL]

It creates a directory with the project. The sources are yours now, master.

## Forking project

Your very first step is to fork a project. Forking means creating your own copy of the project. On GitHub it's so easy with the Fork button in the upper-right corner. Click it and select the account you want the fork go.

In the terminal, go to the project's directory and add the repository as a remote repository.

    git remote add [remote-name] [clone URL]

I tend to use my first name for `remote-name` so I know that my personal repository copy is under `jacek` nick.

## Branching project

Developing a change for a project is the real thing. It can be a documentation page, a fix for an issue or whatever else the project holds.

The following command

    git checkout -b [branch-name]

creates and changes your current branch from `master` (usually) to `branch-name`. Use `wip/` in the `branch-name` to denote that the work is in progress so people can review the changes before they get *squashed* and merged with the master.

## Committing changes to project

On a branch, go the following to commit the changes of yours:

    git commit -m [commit-message] -a

There are some strict rules on how to write a proper `commit-message`. For now, don't worry about it too much. There are tougher things you will have to go through and writing proper commit messages don't belong to that category...yet. It's more important to get you up to speed with contributing to a project than to do it without mistakes from the day 0.

## Pushing changes to remote repo

With the changes on the branch committed, it's time to show off on GitHub. Push the changes with the following command:

    git push [remote-name] [branch-name]

Using command completion can save you a lot of typing here. A decent shell like [oh-my-zsh](http://ohmyz.sh/) is highly recommended (on Mac OS X at the very least).

`remote-name` is the nick of the remote repository, e.g. `jacek` while `branch-name` is the name of the branch you're working on right now.

## Creating pull request on GitHub

With the changes in the remote repository on GitHub, you should now be able to send a pull request to the original repo.

GitHub shows the Pull Request button when you're changes hit your repository that's a fork of the project. Click the button and fill out the blanks. GitHub uses your commit message as the title that further easies the process.

Click Create and you've just contributed to the project! Open Source Contributor badge unlocked! Congratulations.

## Squashing changes

There might be times when your work in progress generates a stream of changes to the branch. You may face a request *to squash the changes* so it goes (aka gets merged) to the master as a single change/commit.

Use `git rebase -i`:

    git rebase -i master

where `master` is the name of the master branch of the project you forked and then branched for your changes.

Fix any merge issues while rebasing. When fixed, `git add` the files and `git rebase --continue` afterwards.

You can always go back to the previous state (before squashing) with `git rebase --abort`.

Doing squashing is worth the time since merging the changes with the master later on becomes a no-brainer for the project maintainers.

You may also want to read the article [squashing commits with rebase](http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html) that has helped me a lot to get the gist of rebasing under git.

## Deleting remote and local branches

When the work is over and all the changes are merged with the master, you can safely delete remote and local branches.

Once the work gets merged, GitHub asks you to delete the branch. Click the button under the pull request.

Delete the local branch with the command:

    git branch -D [branch-name]

where `branch-name` is the name of the branch you want to delete.

You should change the branch to some other branch to be able to delete it.
