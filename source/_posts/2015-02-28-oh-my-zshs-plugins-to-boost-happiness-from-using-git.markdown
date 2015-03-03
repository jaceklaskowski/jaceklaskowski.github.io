---
layout: post
title: "Oh My Zsh's plugins to boost happiness from using git"
date: 2015-02-28 20:26:40 +0100
comments: true
categories: [git,zsh,wip]
sidebar: collapse
keywords: git, zsh, command line
published: true
---
From the website of the very useful and must-have addition to your terminal - [Oh-My-Zsh](http://ohmyz.sh/):

> Oh-My-Zsh is an open source, community-driven framework for managing your ZSH configuration. It comes bundled with a ton of helpful functions, helpers, plugins, themes, and a few things that make you shout...**"Oh My ZSH!"**

As I'm using the ZSH configuration framework so often that I can't believe I could've lived without it for so long here's a collection of the plugins to cut the number of keystrokes while working with git repositories. To my great surprise there are quite a few plugins for git and the list below is my humble summary to get it remembered as much as to spread a word (as I've been meeting people unaware of Oh-My-Zsh far too often).

**FIXME** Consider the blog post complete after the line's gone. It's always in readable state, though.

<!-- more -->

## Oh Mom, there are 10 plugins for git!

I was very surprised to have been presented with **10** plugins for git when I hit `TAB` to complete the `~/.oh-my-zsh/plugins/git` path:

* **git** = (quoting README.md) *this plugin adds several git aliases and increase the completion function provided by zsh*
* **git-extras** = ??? (*aka* unsure what it does - yet to be discovered)
* **git-flow** = (quoting the plugin's sources) *git-flow completion nirvana*
* **git-flow-avh** = unsure how it's different from the `git-flow` plugin
* **git-hubflow** = unsure how it's different from the `git-flow` plugin
* **git-prompt** = (quoting the plugin's sources) *ZSH Git Prompt Plugin*
* **git-remote-branch** = ??? (*aka* unsure what it does - yet to be discovered)
* **gitfast** = ??? (*aka* unsure what it does - yet to be discovered)
* **github** = ??? (*aka* unsure what it does - yet to be discovered)
* **gitignore** = ??? (*aka* unsure what it does - yet to be discovered)

I *just* assume they are all plugins to ease my work with git repositories and am going to review them all one by one and *amend* the blog post afterwards.

The commands I've managed to master so far:

* `gco`   = `git checkout`
* `glo`   = `git log --oneline --decorate --color`
* `glgga` = `git log --graph --decorate --all`
* `gcm`   = `git checkout master`
* `gd`    = `git diff`
* `gdc`   = `git diff --cached`
* `gc!`   = `git commit -v --amend`
* `gcp`   = `git cherry-pick`
* `gl`    = `git pull`
* `gp`    = `git push`
* `gst`   = `git status`
* `grba`  = `git rebase --abort`
* `grbi`  = `git rebase -i`
* `grbc`  = `git rebase --continue`
* `grh`   = `git reset HEAD`
* `grhh`  = `git reset HEAD --hard`
* `grv`   = `git remote -v`

Read the official [Plugins](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins) page to learn how to enable plugins in your configuration.

The configuration of mine includes the following plugins, `git` including:

	➜  ~  grep -e "plugins=(" ~/.zshrc | grep -e "^[^#]"
	plugins=(git osx brew common-aliases)

I'm totally aware that there's plenty of room for improvement here. Let me know what I'm missing (and where I'm wasting my time still). I'd wholeheartedly appreciate any time savings.
