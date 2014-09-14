---
layout: post
title: "How much one ought to know eta expansion"
date: 2013-11-23 22:20:23 +0100
comments: true
categories: scala
sidebar: collapse
---
Have you ever wondered how much you should understand the concept of **eta expansion** to master [Scala](http://scala-lang.org/)? I have and hence the page in which I'm going to collect the wisdom (dug out of Internet) with its applications.

<!-- more -->

**NOTE**: The page is supposed to be a Wiki page not a blog post. It's therefore subject to change without notice. When I find out how to use [Jekyll](http://jekyllrb.com/) and [GitHub Pages](http://pages.github.com/) to power a Wiki-like system, I'll migrate it there right away.

## Eta expansion? Uh, oh, are we in trouble?

Quoting the answer to [What is the difference between a method and a function](http://stackoverflow.com/a/155655/1305344) on StackOverflow:

> A function is a piece of code that is called by name. It can be passed data to operate on (ie. the parameters) and can optionally return data (the return value).
>
> All data that is passed to a function is explicitly passed.
> 
> A method is a piece of code that is called by name that is associated with an object. In most respects it is identical to a function except for two key differences.
>
> 1\. It is implicitly passed the object for which it was called
>
> 2\. It is able to operate on data that is contained within the class (remembering that an object is an instance of a class - the class is the definition, the object is an instance of that data)"

Let's start with the following example.

	scala> def foo = 5
	foo: Int

	scala> val f = foo _
	f: () => Int = <function0>

Do you happen to know the name of *the (implicit) conversion* when the underscore is applied to a method (as if it stood for its parameters) and you eventually end up with a value - *a function* (that some call *a function value*)? That's *eta expansion* which is a means of coercing (converting) methods into functions in Scala.

`f` is a value of `() => Int` type or an instance of `<function0>`. It may have been written as:

	scala> def foo() = 5
	foo: ()Int

Almost.

Can you spot the difference between these three definitions: `def foo = 5` and `def foo() = 5` and `val f = foo _`?

In Scala, **a function** is *always* a (object) value of a function type that's a syntactic sugar for a `Function` trait, e.g. `Int => Boolean` corresponds to Function1[Int, Boolean]. As a value the function value can be assigned to a name, i.e. can be named.

On the other hand, **a method** of a class is *always* a member of the type the class represents and as such doesn't have its own type. It has a signature, though, e.g. `()Int` that looks like a function type, e.g. `() => Int`.

A method *always* belongs to an instance of a class yet looks similar to a function signature-wise.

Think of a method, say `foo`, that accepts a value of a function type.

	scala> def foo(bar: () => Int) = bar()
	baz: (bar: () => Int)Int

	scala> object A {
	     |   def baz() = 5
	     | }
	defined module A

	scala> foo(A.baz)
	res13: Int = 5

	scala> val ff = A.baz _
	ff: () => Int = <function0>

	scala> foo(ff)
	res14: Int = 5

During compilation the call `foo(A.baz)` was transformed to a call to a function value that in turn calls the method `baz` on the `A` sole object. It may have been written as:

	scala> val fv: () => Int = () => A.baz()
	fv: () => Int = <function0>

	scala> foo(fv)
	res15: Int = 5

The function value `fv` is of `Function0[Int]` type.

	scala> val fv: Function0[Int] = new Function0[Int] {
	     |   def apply() = A.baz()
	     | }
	fv: () => Int = <function0>

	scala> foo(fv)
	res17: Int = 5

Quoting the answer to [When do I have to treat my methods as partially applied functions in Scala?](http://stackoverflow.com/a/6651099/1305344) on StackOverflow:

> You have to write the `_` whenever the compiler is not explicitly expecting a `Function` object.

	scala> def foo(n: Int) = n
	foo: (n: Int)Int

	scala> val bar = foo
	<console>:8: error: missing arguments for method foo;
	follow this method with `_' if you want to treat it as a partially applied function
	       val bar = foo
	                 ^

	scala> val bar: Int => Int = foo
	bar: Int => Int = <function1>

## Uniform access principle

Interestingly, uniform access principle makes understanding eta expansion a bit tricker.

	scala> object A {
	     |   def foo(bar: () => Int) = 1
	     |   def foo(n: Int) = 2
	     | }
	defined module A

	scala> def f() = 4
	f: ()Int

	scala> def baz() = 4
	baz: ()Int

Guess what happens when you execute `A.foo(baz)`.

	scala> A.foo(baz)
	res0: Int = 2

What about calling `A.foo` with `baz _`?

	scala> A.foo(baz _)
	res1: Int = 1

Can you explain why? Visit [Eta-expansion between methods and functions with overloaded methods in Scala][1] on StackOverflow for more information.

[1]: http://stackoverflow.com/q/17324247/1305344 "Eta-expansion between methods and functions with overloaded methods in Scala"

There's another example that is not easy to explain.

	scala> def fm: Map[Int, String] = Map(0 -> "zero")
	fm: Map[Int,String]

	scala> fm.isInstanceOf[Function1[Int, String]]
	res15: Boolean = true

	scala> def g: Int => String = fm
	g: Int => String

Since `fm` is of the `Map[Int, String]` type and [scala.collection.immutable.Map](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Map) inherits from [scala.Function1](http://www.scala-lang.org/api/current/index.html#scala.Function1), one can assign `fm` to `g`. That works fine.

Can you explain why the following sample doesn't work?

	scala> def fm(): Map[Int, String] = Map(0 -> "zero")
	fm: ()Map[Int,String]

	scala> def g: Int => String = fm
	<console>:8: error: type mismatch;
	 found   : () => Map[Int,String]
	 required: Int => String
	       def g: Int => String = fm
	                              ^

Note that `fm` got merely the brackets and they're indeed equal.

	scala> fm() == fm
	res16: Boolean = true

See [SI-7187 eta expansion should not precede empty application][2].

[2]: https://issues.scala-lang.org/browse/SI-7187

## Eta expansion and overloaded methods

When there's an object of a type with overloaded methods the underscore `_` may yield different results.

It may resolve to a one-or-more-argument method.

	scala> object A {
	     |   def foo = 5
	     |   def foo(n: Int) = 10
	     | }
	defined module A

	scala> A.foo _
	res6: () => Int = <function0>

The underscore insists on specifying the type of a function value when there's ambiguity regarding the method it should be applied to. And it doesn't matter what the order of parameter types in the overloaded methods is.

	scala> object A {
	     |   def foo(n: Int) = 5
	     |   def foo(s: String, n: Int) = 10
	     | }
	defined module A

	scala> A.foo _
	<console>:9: error: ambiguous reference to overloaded definition,
	both method foo in object A of type (n: Int, m: Int)Int
	and  method foo in object A of type (n: Int)Int
	match expected type ?
	              A.foo _
	                ^

	scala> A.foo _ : ((Int, Int) => Int)
	res5: (Int, Int) => Int = <function2>

Interestingly, the underscore is more forgiving (regarding the explicit type of the function value) when there are two overloaded methods of which one accepts a single parameter of the type [AnyRef](http://www.scala-lang.org/api/current/scala/AnyRef.html). In such case, the other method - the no-AnyRef method is chosen.

	scala> object A {
	     |   def foo(a: AnyRef) = 5
	     |   def foo(s: String, n: Int) = 10
	     | }
	defined module A

	scala> A.foo _
	res5: (String, Int) => Int = <function2>

## Use case - Abstracting over arity

With eta expansion you may forget about the arity of a method.

	scala> class A {
	     |   def foo(x: Int, y: Int, z: Int) = 5
	     | }
	defined class A

	scala> def bar(x: A) = x.foo _
	bar: (x: A)(Int, Int, Int) => Int

It's certainly far simpler (and still type-safe) than doing it explicitly.

	scala> def bar(x: A) = x.foo(_, _, _)
	bar: (x: A)(Int, Int, Int) => Int

When there is another three-argument method in the `A` class, you'd need to specify the parameter types explicitly that makes using the approach even more troublesome.

	scala> def bar(x: A) = x.foo(_: Int, _: Int, _: Int)
	bar: (x: A)(Int, Int, Int) => Int

## Scala Specification about Eta Expansion

In **6.26.5 Eta Expansion** [Scala Language Specification (PDF)](http://scala-lang.org/files/archive/nightly/pdfs/ScalaReference.pdf) says:

> Eta-expansion converts an expression of method type to an equivalent expression
of function type.

The section belongs to **6.26 Implicit Conversions** that gives another perspective on what eta expansion is - it's *an implicit converstion*.

In **6.26 Implicit Conversions** the Scala Language Specification says:

> Implicit conversions can be applied to expressions whose type does not match their
expected type, to qualifiers in selections, and to unapplied methods.

It goes further into when a type is compatible to another.

> We say, a type T is compatible to a type U if T conforms to U after applying eta-expansion (§6.26.5) and view applications (§7.3).

In **6.26.2 Method Conversions** the Scala Language Specification says:

> **Eta Expansion.** Otherwise, if the method is not a constructor, and the expected
type pt is a function type (Ts') => T', eta-expansion (§6.26.5) is performed on the
expression e.

## References (where wisdom came from)

* [Functional Scala: Turning Methods into Functions (or WTF is eta expansion?)](http://gleichmann.wordpress.com/2011/01/09/functional-scala-turning-methods-into-functions/)
* [Eta expansion, meet overloading](https://groups.google.com/d/msg/scala-language/FrDJiagB8CY/3NVmhdKe0vgJ)
* [Scala Language Specification (PDF)](http://scala-lang.org/files/archive/nightly/pdfs/ScalaReference.pdf)
* [Eta-expansion between methods and functions with overloaded methods in Scala](http://stackoverflow.com/q/17324247/1305344)
* [Methods are not Functions](http://tpolecat.github.io/2014/06/09/methods-functions.html)
