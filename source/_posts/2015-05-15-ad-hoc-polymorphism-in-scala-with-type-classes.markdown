---
layout: post
title: "ad hoc polymorphism in Scala with type classes"
date: 2015-05-15 22:21:20 +0200
comments: true
categories: [scala]
sidebar: collapse
keywords: polymorphism, scala, typeclasses, design pattern
published: true
---

My journey into the depths of [Scala](http://www.scala-lang.org/) is in full swing. Not only can I learn the theory (with the group of [Warsaw Scala Enthusiasts](http://warsawscala.pl)), but also apply it to commercial projects (with the Scala development teams of [DeepSense.io](http://deepsense.io/) and [HCore](http://www.hcore.com/)). Each day I feel I'm getting better at using **type system in Scala** in a more concious and (hopefully) efficient manner.

This time I sank into **type classes** that is a means of doing ** *ad hoc* polymorphism** in Scala.

From [*ad hoc* polymorphism](http://en.wikipedia.org/wiki/Ad_hoc_polymorphism) article on Wikipedia:

> In programming languages, **ad hoc polymorphism** is a kind of polymorphism in which polymorphic functions can be applied to arguments of different types, because a polymorphic function can denote a number of distinct and potentially heterogeneous implementations depending on the type of argument(s) to which it is applied.

The blog post presents a way to implement the type classes concept in Scala.

p.s. I'm yet to find out how much of it is [multimethods](http://clojure.org/multimethods) in [Clojure](http://clojure.org/) (that was once of much help to introduce me to **functional programming**).

<!-- more -->

## Theory

From the article [Polymorphism (computer science)](http://en.wikipedia.org/wiki/Polymorphism_(computer_science)) on Wikipedia, you can read about three different kinds of polymorphism:

* **subtyping**
* **parametric polymorphism**
* **ad hoc polymorphism**

The order does matter (and is different from the Wikipedia article's one) as I think that's exactly the order of when and how well developers master them (in any programming language).

I reckon the Scala community finds the first two quite often used in code that contributes to how well it's understood and applied (except [variance](http://en.wikipedia.org/wiki/Covariance_and_contravariance_%28computer_science%29)), and just the last one - ad hoc polymorphism - is proving a major hurdle for many, me including. It didn't click for me for a long time, either, only until I found the "venues" of very concise and comprehensible material (see [References](#references) section below).

> In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types.

The difference between *subtyping* and *parametric polymorphism* and *ad hoc polymorphism* is that the type hierarchy is expressed explicitly using **extends** keyword in Scala (for subtyping) or type parameters (for parametric polymorphism) while ad hoc polymorphism bases itself on **implicit classes** to *mixin* the behaviour (using traits). And that's pretty much all, technically.

Let's see it in a Scala code.

## Practice

### Algebraic data types (raw data)

Let's assume you have the following type hierarchy (from [Tutorial: Typeclasses in Scala with Dan Rosen](https://youtu.be/sVMES4RZF-8)):

    sealed trait Expression
    case class Number(value: Int) extends Expression
    case class Plus(lhs: Expression, rhs: Expression) extends Expression
    case class Minus(lhs: Expression, rhs: Expression) extends Expression

No behaviour, but *algebraic data types* or (often called) *raw data*.

Think about how you'd go about evaluating expressions, i.e. `Plus(Minus(Number(5), Number(3)), Number(18))`, i.e. how to make the following application compile and print `20`?

    object Main extends App {
      val expr = Plus(Minus(Number(5), Number(3)), Number(18))
      println(...) // do something with expr so it prints 20
    }

The past me would change `sealed trait Expression` to include `def value: Int` and force the three case classes `Number`, `Plus` and `Minus` follow. I'd not be surprised if you thought exactly so. You should not, however.

Think about the case where you must **not** change them as they could be provided as a library or be part of the language or be licensed or...I simply ask you not to.

### objects to apply behaviour

Since you're in Scala, you could easily work it around with an object, say `object ExpressionEvaluator`, that would *pattern match* to types and do the heavy lifting, i.e. know what to do with each and every type:

    object ExpressionEvaluator {
      def value(expression: Expression): Int = expression match {
        case Number(value) => value
        case Plus(lhs, rhs) => value(lhs) + value(rhs)
        case Minus(lhs, rhs) => value(lhs) - value(rhs)
      }
    }

That's quite inefficient and cumbersome, though. You'd have to know about all of the implementations as well as about what to do for each. It's nearly impossible to have right, complete and still flexible.

With the above object, you'd write:

    object Main extends App {
      val expr = Plus(Minus(Number(5), Number(3)), Number(18))
      println(ExpressionEvaluator.value(expr)) // print the value
    }

### implicits in Scala

Let's do it the right way, so it's not `ExpressionEvaluator` to know what to do with every `Expression`, but expressions themselves do what they're supposed to do instead.

You may have heard about **implicits** in Scala. Perhaps, you may have used them, too. Think about a solution with implicit machinery so the following is possible:

    object Main extends App {
      val expr = Plus(Minus(Number(5), Number(3)), Number(18))
      println(expr.value) // print the value
    }

One way in Scala would be to apply **Pimp my Library** pattern leveraging `implicit class`es to add necessary methods as follows:

    implicit class ExpressionOps(e: Expression) {
      def value = ... // calculate the value
    }

And have a `implicit class` per `case class` and `sealed trait Expression`, too. The reason to have the implicits is to add a `value` method to every type, i.e. implicitly convert types without `value` to ones that have it.

    implicit class ExpressionOps(e: Expression) {
      def value: Int = e match {
        case n : Number => n.value
        case p : Plus => p.value
        case m : Minus => m.value
      }
    }

    implicit class PlusOps(p: Plus) {
      def value: Int = p.lhs.value + p.rhs.value
    }

    implicit class MinusOps(m: Minus) {
      def value: Int = m.lhs.value - m.rhs.value
    }

Note that since `Number` had `value` already, an implicit was not needed.

With the implicits in place, you can write:

    println(expr.value)  // prints 20

The implicit-based solution is far much more flexible because calculating a value is the responsibility of an implicit in scope -- a change in a single implicit, say `MinusOps`, would only change how `Minus.value` works (with no changes to the rest of the "framework").

You're halfway to typeclasses!

### Partial ad hoc polymorphism

Think about the case where you'd have a library to calculate a value of `Valueable` values (no pun intended). Say, you have such a library that offers the following "calculator":

    def calculate(v: Valueable) = ... // calculate the value of v

How would you go about converting `Expression`s to `Valueable`s so a value of `Expression` type would participate in the contract of `def calculate`? 

You're right that whenever type conversion is needed in Scala, implicits have their say -- you used them already to have `def value` for `Expression` type hierarchy. You're going to use them again with a very small change that has enormous impact. Doing so will introduce the type class design pattern.

The current solution relies on values with `def value` -- it's like having a library that uses structural typing in Scala that unfortunatelly uses reflection and hence is very costly performance-wise. Happily, you don't need structural types.

If you had a library that expects values of some type, say `trait Valueable`, to calculate a value or some other library to JSONify them (using some `trait Json`), the previous solutions would fall short -- you've merely met the requirement of being able to call a `value` method on values, but the values don't belong to a single type hierarchy of some `trait Valueable` that the library uses.

Assume you have a library that works with `trait Valueable`-only values and there's a `calculate` method to work with them as follows:

    trait Valueable {
      def value: Int
    }

    def calculate(v: Valueable) = v.value

Note that the library knows nothing about the `Expression` type hierarchy. The `Expression` type hierarchy could not even have existed at the time `Valueable` did.

A more flexible and efficient solution is to *somehow* meld the `Expression` type hierarchy with `Valueable` and create *is-a relationship*. No, no, you're not going to change the `Expression` trait in any way. You must **not** and you even **can't**, remember?

Welcome to **typeclasses** (also known as **type class design pattern**)!

In order to have `extends`-like semantic at runtime with no `extends Valueable` in the (closed and `sealed`) `Expression` type hierarchy you change `implicit class`es to have the common trait mixed in, instead.

    implicit class ExpressionOps(e: Expression) extends Valueable {
      def value = e match {
        case n: Number => n.value
        case p: Plus => p.value
        case m: Minus => m.value
      }
    }

    implicit class PlusOps(p: Plus) {
      def value: Int = p.lhs.value + p.rhs.value
    }

    implicit class MinusOps(m: Minus) {
      def value: Int = m.lhs.value - m.rhs.value
    }

With the above `implicit class`es that all `extends Valueable`, you can safely do `expr.value` for any `Expression` value for which an implicit exists in scope and be done with the task at hand. Notice how Scala "executes" `value` on the different types (that's based upon using `implicit class`es for the types when `value` is needed).

    println(calculate(expr))  // prints 20

For what is worth, the `calculate` method belongs to a library that knows nothing about `Expression` type and you can't change it so `Expression`s would be worked with. That's exactly where typeclasses shines and are hard to beat.

This is the version of type classes pattern as described in the book [Programming Scala, 2nd Edition](http://shop.oreilly.com/product/0636920033073.do) in "Type Class Pattern" section, page 156.

### Final solution: `Value[T]` type class

You can still do better than that in Scala and that's what (*the real*) type class design pattern offers. This is the version of type class pattern as demonstrated in the video [Tutorial: Typeclasses in Scala with Dan Rosen](https://youtu.be/sVMES4RZF-8). You should really watch it.

Let's have a parameterized type `trait Value[T]` that's supposed to "bridge" the type hierarchies - `Expression` and `Valueable`:

    trait Value[T] {
      def value(t: T): Valueable
    }

The fictitious Calculator library has `object Calculator` with a single `def calculate(v: Valueable): Int` method:

    object Calculator {
      def calculate(v: Valueable): Int = v.value
    }

No `Expression`s, just `Valueable`s. That's where the type class pattern shines.

Create `object CalculatorEx` as follows:

    object CalculatorEx {
      def calculate[T : Value](t: T): Int =
        Calculator.calculate(implicitly[Value[T]].value(t))
    }

It says that for any type `T` there's an implicit conversion to a `Value[_]` type hierarchy using implicits (that are used in the method via [implicitly](http://www.scala-lang.org/api/current/index.html#scala.Predef$)).

All in all, you need to throw in three more implicits for `Number`, `Plus` and `Minus` so they can be seen as `Valueable` and participate in `def calculate(v: Valueable): Int`-based library:

    implicit val number2Value = new Value[Number] {
      def value(n: Number): Valueable = new Valueable {
        override def value: Int = n.value
      }
    }

    implicit val plus2Value = new Value[Plus] {
      def value(p: Plus): Valueable = new Valueable {
        override def value: Int = p.lhs.value + p.rhs.value
      }
    }

    implicit val minus2Value = new Value[Minus] {
      def value(m: Minus): Valueable = new Valueable {
        override def value: Int = m.lhs.value - m.rhs.value
      }
    }

These make it possible to calculate the value of the expression leveraging the fictitious Calculator library:

    println(CalculatorEx.calculate(expr))  // prints 20

That's it! You're done. If you've followed along closely and have developed the "framework" on your own, you should have a complete understanding of the type class design pattern in Scala. Without the type class pattern, blending the `Expression` type hierarchy with `Valueable` would not have been possible! And that was the goal of the exercise.

### Follow up - All `Valueable`

Think about using other types, say `Int`, with the fictitious Calculator library so the following is possible:

    println(CalculatorEx.calculate(1))  // prints 1

As the Scala compiler says:

> could not find implicit value for evidence parameter of type Value[Int]

All, we need to do to blend `Int`s as `Valueable`s is to have an implicit that does the conversion.

    implicit val int2Value = new Value[Int] {
      override def value(t: Int) = new Valueable {
        override def value: Int = t
      }
    }

And that's it!

As a final exercise would be to reuse implicits and create more complex ones to support "sum" types, like `Tuple`. I'm leaving it as a homework. Have fun!

Let me know how the homework and the whole blog post went out in the Comments section below. I'd appreciate any comments to improve upon.

## References

There are plenty of very good sources on the topic of type classes in general and in Scala, in particular, but the following have done wonders for me:

* [ad hoc polymorphism](http://en.wikipedia.org/wiki/Ad_hoc_polymorphism) article on Wikipedia
* [Polymorphism (computer science)](http://en.wikipedia.org/wiki/Polymorphism_(computer_science)) article on Wikipedia
* [Tutorial: Typeclasses in Scala with Dan Rosen](https://youtu.be/sVMES4RZF-8) video
* [Programming Scala, 2nd Edition](http://shop.oreilly.com/product/0636920033073.do) book