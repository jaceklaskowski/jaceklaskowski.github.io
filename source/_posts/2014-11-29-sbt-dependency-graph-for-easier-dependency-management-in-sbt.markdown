---
layout: post
title: "sbt-dependency-graph for easier dependency management in sbt"
date: 2014-11-29 23:10:06 +0100
comments: true
categories: [Scala, sbt]
sidebar: collapse
keywords: scala
published: true
---
That's gonna be short and hopefully simple. If you're with [sbt](http://www.scala-sbt.org/) you're going to like it very much.

Edit `~/.sbt/0.13/plugins/sbt-dependency-graph.sbt` so it looks as follows:

	addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.7.4")

Edit `~/.sbt/0.13/global.sbt` so it looks:

	net.virtualvoid.sbt.graph.Plugin.graphSettings

With these two files, open `sbt` or `activator` and execute `dependencyGraph` (I used [hello-slick-specs2](https://github.com/jaceklaskowski/hello-slick-specs2) project):

	> dependencyGraph
	[info] Updating {file:/Users/jacek/dev/oss/hello-slick-specs2/}hello-slick-specs2...
	[info] Resolving jline#jline;2.12 ...
	[info] Done updating.
	[info]                             +---------------------------+
	[info]                             |hello-slick-specs2_2.11 [S]|
	[info]                             |          default          |
	[info]                             |            1.0            |
	[info]                             +---------------------------+
	[info]                                    |     |   |    |
	[info]                ---------------------     |   |    ----------------------------
	[info]                |                         |   -----------------               |
	[info]                v                         v                   |               |
	[info]           +---------+          +------------------+          |               |
	[info]           |slf4j-nop|          |  slick_2.11 [S]  |          |               |
	[info]           |org.slf4j|          |com.typesafe.slick|          |               |
	[info]           |  1.7.7  |          |      2.1.0       |          |               |
	[info]           +---------+          +------------------+          |               |
	[info]                |                   |   ||      |             |               |
	[info]      -----------                   |   ||      ---------     |               |
	[info]      |  ----------------------------   ||              |     |               |
	[info]      |  |             ------------------|              |     |               |
	[info]      |  |             |                 |              |     |               |
	[info]      v  v             v                 v              v     v               v
	[info]  +---------+ +-----------------+ +------------+ +------------------+ +--------------+
	[info]  |slf4j-api| |    slf4j-api    | |   config   | |  scala-library   | |      h2      |
	[info]  |org.slf4j| |    org.slf4j    | |com.typesafe| |  org.scala-lang  | |com.h2database|
	[info]  |  1.7.7  | |      1.6.4      | |   1.2.1    | |      2.11.1      | |   1.4.182    |
	[info]  +---------+ |evicted by: 1.7.7| +------------+ |evicted by: 2.11.4| +--------------+
	[info]              +-----------------+                +------------------+
	[info] Note: The old tree layout is still available by using `dependency-tree`
	[success] Total time: 0 s, completed Nov 29, 2014 11:19:30 PM

Neat, isn't it?

You may also want to execute `dependencyGraphMl`:

	> dependencyGraphMl
	[info] Wrote dependency graph to '/Users/jacek/dev/oss/hello-slick-specs2/target/dependencies-compile.graphml'
	[success] Total time: 0 s, completed Nov 29, 2014 11:21:46 PM

Install [yEd](http://www.yworks.com/en/products/yfiles/yed/) and open the graph:

	> eval "open target/dependencies-compile.graphml" !
	[info] ans: Int = 0

{% img /images/hello-slick-specs2-yed-graph.png yEd graph of compile dependencies %}

I really wish I'd known it earlier. It'd surely have saved me a lot of time.