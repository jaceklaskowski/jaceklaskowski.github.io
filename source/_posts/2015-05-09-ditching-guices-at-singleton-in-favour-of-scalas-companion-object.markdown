---
layout: post
title: "Ditching Guice's @Singleton in favour of Scala's (companion) object"
date: 2015-05-09 14:57:19 +0200
comments: true
categories: [scala]
sidebar: collapse
keywords: guice, singleton pattern, companion object
published: true
---

[Arek Komarzewski](https://twitter.com/akomarzewski) (a Scala developer in HCore) mentioned the following this Friday and made my day (and the whole week, too):

> I can now ditch Guice's @Singleton as I've got a trait and the companion object combo (thanks to Scala).

This time the blog post is without a complete working example. Not yet. It's to remind myself to prepare one (or be given one after the blog post is published -- whatever comes first). I just think it needs to be said aloud to be heard and think about.

<!-- more -->

And then, quite unexpectedly to me, [Play Framework](https://www.playframework.com/) - *"The High Velocity Web Framework For Java and Scala"* - that's the web framework supported by Typesafe in their [Reactive Platform](http://www.typesafe.com/products/typesafe-reactive-platform) has announced in [What's new in Play 2.4](https://www.playframework.com/documentation/2.4.x/Highlights24):

> In the Scala ecosystem, the approach to dependency injection is not generally agreed upon, with many competing compile time and runtime dependency injection approaches out there.

> Play’s philosophy in providing a dependency injection solution is to be unopinionated in what approaches we allow, but to be opinionated to the approach that we document and provide out of the box. For this reason, we have provided the following:

> An implementation that uses Guice out of the box

I'm very lucky to be able to pursue my understanding of Scala the programming language not only in my free time, but also in commercial projects as a full-time Scala developer and a technical leader (in [DeepSense.io](http://deepsense.io/)) as well as supporting companies making the most out of Scala and open source software (in [HCore](http://www.hcore.com/)) not to mention leading [Warsaw Scala Enthusiasts](http://www.meetup.com/WarszawScaLa/) in **Warsaw**, **Poland**. The technical part of my life simply can't be any better! I'm learning as well as teaching people using Scala as an object-oriented and functional programming language on JVM, and am also meeting up lots of Scala developers. I really wish I had more time to publish all the major breakthroughs in blog posts here.

Enough praising. On to real matters.

In [DeepSense.io](http://deepsense.io/) we're using [Guice](https://github.com/google/guice) as *a dependency framework*. It's also used in few other projects where Scala is used. That often leads to my questioning the need for Guice or any dependency injection framework since Scala the programming language itself offers enough features to let Guice leave.

I think there's the issue of how most Scala developers (I'm meeting) think -- they are former Java developers who see no reason to adapt to new approaches of tackling development problems. They simply have no more courage to dig deeper, and...let the past rest and welcome new solutions for the brighter future. And since Scala alone is still shaping itself and the Scala community is not clear on what to follow along and what to forget about, that all makes the leave-past-welcome-new approach harder.

Just this week I had a pleasure to meet two teams that both use Guice because nobody introduced viable alternatives (even when they use one already - Scala!). I don't consider myself skilled enough, either, since I'm pretty much a Guice newbie and hence just unable to counter the Guice's way of solving problems. It's that something inside me that is telling me that Guice is dragged along unnecessarily and gives a false perception of being the right fit to dependency injection-looking problems (even when it introduces more problems, mostly related to learning a yet another framework, than it solves). I'm glad Spring is far heavier as it would likely have found its way in the projects, too.

It has worked fine for Arek and I'm hoping it's going to work fine for me soon, too. I'll simply have to find it out and promote the right approach.
