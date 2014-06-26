---
layout: post
title: "Beginner \"guide\" to sbt 0.13 and IntelliJ IDEA 13"
date: 2013-12-07 16:39:37 +0100
comments: true
categories: scala sbt intellij-idea
sidebar: collapse
---
{% img left /images/intellij-idea-13-new-logo.png The logo of IntelliJ IDEA 13 %} It has not been very long ago when the only way to work with [sbt](http://www.scala-sbt.org/) projects in [IntelliJ IDEA](http://www.jetbrains.com/idea/) was to use the [sbt-idea](https://github.com/mpeltonen/sbt-idea) plugin that aimed at *"creating IntelliJ IDEA project files"*.

[With the recent release of IntelliJ IDEA 13](http://www.jetbrains.com/idea/whatsnew/) (build: 133.193,  released: December 3, 2013) it's no longer true - the version comes with built-in sbt support and the support is available in [the free Community Edition](http://www.jetbrains.com/idea/download/index.html), too.

My recent, rather quite frequent visits on [StackOverflow](http://stackoverflow.com/) have showed that there's one question very often asked - **How to start using sbt with IntelliJ IDEA?** It turns out that the latest version of IntelliJ IDEA 13 squashed it pretty neatly and the built-in sbt support made the question irrelevant.

Unless it changes, the page remains empty[^1]. What a surprise, isn't it? I've always been thinking about writing the best beginner guide and here it is...at long last!

On to querying [StackOverflow's #sbt tag](http://stackoverflow.com/questions/tagged/sbt) for questions about *a* sbt support in IntelliJ IDEA 13.

[^1]: as a kind of placeholder for future tips and tricks