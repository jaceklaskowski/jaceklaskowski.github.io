---
layout: post
title: "About Function1.andThen in Scala"
date: 2013-11-26 23:00:59 +0100
comments: true
categories: scala
sidebar: collapse
published: false
---

When you need to compose a function out of a series of smaller, more specialized functions, scala.Function1.andThen() might be an option.

<!-- more -->

**NOTE**: The page is supposed to be a Wiki page not a blog post. It's therefore subject to change without notice. When I find out how to use [Jekyll](http://jekyllrb.com/) and [GitHub Pages](http://pages.github.com/) to power a Wiki-like system, I'll migrate it there right away.

## scala.Function1

Quoting scala.Function1's scaladoc:

> A function of 1 parameter.
