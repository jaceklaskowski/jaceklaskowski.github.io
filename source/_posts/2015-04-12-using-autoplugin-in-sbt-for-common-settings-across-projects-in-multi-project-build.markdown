---
layout: post
title: "Using AutoPlugin in sbt for common settings across projects in multi-project build"
date: 2015-04-12 15:20:45 +0200
comments: true
categories: [sbt, tools]
sidebar: collapse
keywords: autoplugin, sbt
published: true
---

What a joy to learn all the goodies sbt brings to the table and be given a chance to apply it right away to commercial projects in Scala!

I've recently been assigned to a task to create a solution to share common settings across projects in a [multi-project build](http://www.scala-sbt.org/0.13/tutorial/Multi-Project.html) in a Scala project managed by [sbt](http://www.scala-sbt.org). With the new feature of sbt - [autoplugins](http://www.scala-sbt.org/0.13/docs/Plugins.html) - it was very easy to implement from the day one.

<!-- more -->

## The solution = project/CommonSettingsPlugin.scala

	import sbt._
	import Keys._

	object CommonSettingsPlugin extends AutoPlugin {
	  override def trigger = allRequirements
	  override lazy val projectSettings = Seq(
	    organization  := "io.deepsense",
	    version       := "0.1.0",
	    scalaVersion  := "2.11.6",
	    scalacOptions := Seq(
	      "-unchecked", "-deprecation", "-encoding", "utf8", "-feature",
	      "-language:existentials", "-language:implicitConversions"
	    )
	  )
	}

Since the autoplugin is automatically triggered for all the projects in the multi-project build -- `override def trigger = allRequirements` -- all the projects have `organization`, `version`, `scalaVersion`, `scalacOptions` set. No other work is needed.

Simple and easy, and, what's tracked under another issue, Jenkins should now be able to execute `coverageReport` and `coverageAggregate` without troubles for the project!

As a bonus, you don't have to change anything in your sbt project to leverage the simplicity - just drop the code as `project/CommonSettingsPlugin.scala` and do `show scalaVersion`. From that moment on, all the (sub)projects in your multi-project build should have the same `scalaVersion` (unless you use a separate `build.sbt` for a project and set `scalaVersion` explicitly).

## Lessons learnt

The initial mistake was to create a separate sbt project just to keep the autoplugin's code.

From the very first day of the plugin's life it was so much different from the other projects in the build -- it almost yelled out in pain at me not to place it where it was initially. It was a sbt plugin and as such it only meant to enhance sbt itself (not be a part of the commercial project).

What's even worse, up to the latest version of sbt 0.13.8 all plugins (are doomed to) use Scala 2.10.4 that's an issue for [sbt-scoverage plugin](https://github.com/scoverage/sbt-scoverage) as it kept refusing to work with the version of Scala.

All in all, I seemed reluctant for a long time to have even thought of a better place for the plugin even though I had actually known it.

A few discussions on [gitter on the sbt channel](https://gitter.im/sbt/sbt) made my day after the very helpful and nice people from the channel lent me a helping hand to find the final solution - the autoplugin went under `project` and...my live was so much bright again!

What was later pointed out to me when we discussed the change in the team, the final solution was a mere 24 line removal (!)

{% img /images/using-autoplugin-patchset-gerrit.png Patchset with AutoPlugin changes %}

It got `+2` from a teammate and has been merged to `master` without much hussle.

p.s. Shhh, the team thinks it was just me to have figured it out myself.
