---
layout: post
title: "Mistakes introducing Slick for database access under Play Framework 2.3.4 and 2.4.0-M1"
date: 2014-09-12 21:47:50 +0200
comments: true
categories: [Play Framework, Slick, Scala]
sidebar: collapse
keywords: lessons learnt,development,play framework,scala,slick,database
published: true
---
This is a summary of my attempt to run [Slick](http://slick.typesafe.com/) under [Play Framework](https://www.playframework.com/) **2.3.4** and **2.4.0-M1** that ultimately turned out a quite successful endeavour.

I’m currently using Play’s Anorm and writing the queries myself didn’t seem to be something I much liked. I’ve been seeing Slick has had a good press so it was a no-brainer to pick it as an alternative.

<!-- more -->

## Mistake #1. Hurting myself with cutting edge versions

Reading the documentation at http://slick.typesafe.com/ was not very informative as there was no word about how to embed it in a Play Framework application. There should really be a "Getting started with Slick and Play Framework" tutorial since both are part of [The Typesafe Reactive Platform](https://typesafe.com/platform).

I added Slick to `build.sbt` as follows:

    "com.typesafe.slick" %% "slick" % “2.1.0"

And then I realised that I’d have had to do:

    Database.forURL("jdbc:h2:mem:test1", driver = "org.h2.Driver") withSession {
      implicit session =>
      // <- write queries here
    }

but Play Framework gives me:

    DB.withConnection("sayenedb") { implicit c =>
        ...
    }

It was certainly not the way to follow. I needed a solution that would read database configuration from Play's.

## Mistake #2. Using play-slick with Play 2.4.0-M1

Time has come to give [play-slick](https://github.com/playframework/play-slick) a serious try. The project aims to make *"Slick a first-class citizen of Play 2.x.”*

It took me a while to abandon the idea of running play-slick with Play 2.4.0-M1 since [this](https://groups.google.com/d/msg/play-framework/m_bxuqgSKgk/Z4WgfUer19wJ):

> The purpose of this release is to get feedback about the approach to dependency injection that we're implementing in Play 2.4.  The old Play plugins mechanism is going to be deprecated.  For a detailed overview of the different styles of DI available in Play 2.4, please read here:

> https://www.playframework.com/documentation/2.4.x/ScalaDependencyInjection
https://www.playframework.com/documentation/2.4.x/ScalaCompileTimeDependencyInjection
https://www.playframework.com/documentation/2.4.x/JavaDependencyInjection

I reported [an issue for play-slick](https://github.com/playframework/play-slick/issues/208) hoping that the developers notice the missing integration point and the support for 2.4 will come...sooner (?)

## Mistake #3. Using custom db configuration in Play

The application uses no `db.default` configuration in `application.conf` in Play - just a custom one. The result?

    [error] application -

    ! @6jg39b0pk - Internal server error, for (GET) [/tips/wgcategories] ->

    play.api.Configuration$$anon$1: Configuration error[Slick error : jdbc driver not defined in application.conf for db.default.driver key]
         at play.api.Configuration$.play$api$Configuration$$configError(Configuration.scala:94) ~[play_2.11-2.3.4.jar:2.3.4]
         at play.api.Configuration.reportError(Configuration.scala:743) ~[play_2.11-2.3.4.jar:2.3.4]
         at play.api.db.slick.Config$$anonfun$1.apply(Config.scala:64) ~[play-slick_2.11-0.8.0.jar:0.8.0]
         at play.api.db.slick.Config$$anonfun$1.apply(Config.scala:64) ~[play-slick_2.11-0.8.0.jar:0.8.0]
         at scala.Option.getOrElse(Option.scala:120) ~[scala-library-2.11.2.jar:na]

I had to add the following entries to work it around:

    db.default.driver=org.postgresql.Driver
    db.default.url=${?DB_CONN}

`DB_CONN` is the property I set up at command line at Play startup.

Using two datasources required following [Advanced drivers configuration](https://github.com/playframework/play-slick/wiki/ScalaSlickDrivers) and applying the (in)famous Cake pattern.

## Mistake #4. Copying examples to production code - PostgreSQL is case sensitive for field names in quotes

I had the following in my Component:

    def id = column[Int]("ID", O.PrimaryKey, O.AutoInc)

That generated a query with `"ID"` in `select` clause that in turn resulted in the following error:

    STATEMENT:  select s13."ID", s13."name" from “xxx" s13;
    ERROR:  column s13.ID does not exist at character 8

[As Rene pointed out in his tweet](https://twitter.com/rgielen/status/510501297473462272):

> @jaceklaskowski not true - #PostgreSQL follows SQL standard. Columns names are case insensitive unless created with quotation marks

He was right - changing `ID` to `id` has indeed fixed the issue.

    def id = column[Int]("id", O.PrimaryKey, O.AutoInc)

## Mistake #5. [SI-3664] Explicit case class companion does not extend Function / override toString

There's a bug in the Scala compiler that stands in a way when `def * = ...` is defined for a case class as follows:

    class Users(tag: Tag) extends Table[User](tag, "users") {
        def id    = column[Int]   ("id", O.PrimaryKey, O.AutoInc)
        def login = column[String]("login")

        def * = (id, login) <> (User.tupled, User.unapply)
    }

For the `Users` class the compiler says:

> value tupled is not a member of object model.User

A solution is in another issue report [#11 Companion object cover method "tupled" in case class](https://github.com/VirtusLab/unicorn/issues/11) and boils down to using the following instead:

    (User.apply _).tupled

The mapping definition would then look as follows:

    class Users(tag: Tag) extends Table[User](tag, "users") {
        def id    = column[Int]   ("id", O.PrimaryKey, O.AutoInc)
        def login = column[String]("login")
    
        def * = (id, login) <> ((User.apply _).tupled, User.unapply)
    }

See [Mapped projection with <> to a case class with companion object in Slick](http://stackoverflow.com/q/15175659/1305344) for alternative solution.

## Mistake #6. JodaTime support

For cases where you need to use JodaTime types in Slick you should use [slick-joda-mapper](https://github.com/tototoshi/slick-joda-mapper#slick-joda-mapper).

Else you have to stick to `java.sql.Date`, `java.sql.Time`, `java.sql.Timestamp` as described in [Table Rows](http://slick.typesafe.com/doc/2.1.0/schemas.html?highlight=date#table-rows) in the Slick documentation.

## Summary

Using a non-default configuration is always a kind of minefield. Stay away from it unless you’re adventurous and have enough time and patience to fix issues along the way. You’ve been warned.
