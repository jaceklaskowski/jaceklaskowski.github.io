---
layout: post
title: "trait Init[Scope] in sbt"
date: 2014-07-22 13:19:54 +0200
comments: true
categories: scala sbt
sidebar: collapse
---
It's been my wish to master [Scala](http://scala-lang.org/) recently and since I've been spending more time with [sbt](http://www.scala-sbt.org/) I've made the decision to use one to master the other (in no particular order). There are quite a few sophisticated projects in Scala out there, but sbt is enough for my needs.

In order to pursue my understanding of sbt (and hence Scala itself) I've been reading the sources that honestly keep surprising me so much often. It's almost every minute when I find myself scratching my head to digest a piece of sbt code. It's akin to when I was reading the source code of [Clojure](http://clojure.org/) to learn the language. People can write complicated code and I wouldn't be surprised to hear sbt's sources belong to the category. I don't care, though. I'm fine with the complexity hoping the mental pain brings me closer to master Scala.

Today I picked the trait [sbt.Init](https://github.com/sbt/sbt/blob/0.13/util/collection/src/main/scala/sbt/Settings.scala#L41) believing it'd be an important step in my journey.

**NOTE** It becomes feature-complete when the note disappears. Live with the few mistakes for now. Let me know what you think in the Comments section. The site is on GitHub so pull requests are warmly welcome, too. Thanks!

<!-- more -->

There’s the trait [sbt.Init](https://github.com/sbt/sbt/blob/0.13/util/collection/src/main/scala/sbt/Settings.scala#L41). I don’t really know what its purpose is and I hope to find it out after few Scala snippets. There’s just enough hope to master Scala while pursuing my understanding of sbt with the trait.

## Goal

Create an instance of trait `Init[Scope]`.

## Solution

```
val init = new Init[Int] {
  def showFullKey: Show[ScopedKey[_]] = Show { (sk: ScopedKey[_]) => 
    s"${sk.scope}:${sk.key}...${sk.scopedKey}"
  }
}
```
Run `sbt` and then execute the command `consoleProject` to open sbt's Scala REPL with all the necessary types of sbt loaded.

## Mental issues encountered

1. I’m far from being able to distinguish easily type parameters, e.g. `Scope`, in parameterised types, e.g. `Init[Scope]`, from types themselves. When I see `Init[Scope]` my Java-trained eyes see `Scope` type within `Init` type and although it doesn’t make sense after a moment that’s my initial thought.

2. The type constructor `Show[ScopedKey[_]]` in the return value type of `showFullKey` is another trait `Show` that comes with `apply` that is supposed to return a `String` instance from `ScopedKey[_]`. But hey, `ScopedKey[_]` is another type constructor, and things get more complex for me again. Happily, `Show` has a companion object with `apply` method. The story ends as `ScopedKey` is a final parameterized case class and the function parameter `f: T => String` in `Show` returns a `String` so I've just merely followed the types and it *happened* to work fine. The Scala compiler happy and so am I.

## Summary

`Show` is a function type (with `apply`) that accepts `T` and returns `String`. In our case, `T` is `ScopedKey[_]` that’s...well...it’s yet to be understood.

## consoleProject in sbt

If you happened to want to see the code in action, execute `sbt consoleProject` and give the following a try:
```
// (attribute) key that points at Int value
scala> val number = AttributeKey[Int]("number", "number stringified")
number: sbt.AttributeKey[Int] = number
 
scala> val init = new Init[Int] {
     |   def showFullKey: Show[ScopedKey[_]] = Show { (sk: ScopedKey[_]) =>
     |     s"${sk.scope}:${sk.key}...${sk.scopedKey}"
     |   }
     | }
init: sbt.Init[Int] = $anon$1@1f95802
 
scala> val sfk: Show[init.ScopedKey[_]] = init.showFullKey
sfk: sbt.Show[init.ScopedKey[_]] = sbt.Show$$anon$1@7f54be72

scala> val s = sfk(init.ScopedKey[Int](scope=5, key=number))
s: String = 5:number...ScopedKey(5,number)
```