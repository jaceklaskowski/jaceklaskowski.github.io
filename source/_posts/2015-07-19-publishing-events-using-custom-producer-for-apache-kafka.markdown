---
layout: post
title: "Publishing events using custom producer for Apache Kafka"
date: 2015-07-19 23:04:21 +0200
comments: true
categories: [apache-kafka, scala]
sidebar: collapse
keywords: apache, kafka, broker, scala
published: true
---
I'm a [Scala](http://www.scala-lang.org/) proponent so when I found out that [the Apache Kafka team has decided to switch to using Java as the main language of the new client API](https://cwiki.apache.org/confluence/display/KAFKA/Client+Rewrite) it was beyond my imagination. [Akka](http://akka.io/docs/)'s fine with their Java/Scala APIs and so I can't believe [Apache Kafka](http://kafka.apache.org/) couldn't offer similar APIs, too. It's even more weird when one finds out that Apache Kafka itself is written in Scala. Why on earth did they decide to do the migration?!

In order to learn Kafka better, I developed a custom producer using the latest Kafka's Producer API in Scala. I built Kafka from the sources, and so I'm using the version **0.8.3-SNAPSHOT**. It was pretty surprising experience, esp. when I ran across  [java.util.concurrent.Future](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html) that seems so limited to what [scala.concurrent.Future](http://docs.scala-lang.org/overviews/core/futures.html) offers. No `map`, `flatMap` or such? So far I consider the switch to using Java for the Client API a big mistake.

Here comes the complete Kafka producer I've developed in Scala that's supposed to serve as a basis for my future development endeavours using the API in what's going to be in 0.8.3 release.

<!-- more -->

## Custom KafkaProducer in Scala

```scala
import java.util.concurrent.Future

import org.apache.kafka.clients.producer.RecordMetadata

object KafkaProducer extends App {

  val topic = util.Try(args(0)).getOrElse("my-topic-test")
  println(s"Connecting to $topic")

  import org.apache.kafka.clients.producer.KafkaProducer

  val props = new java.util.Properties()
  props.put("bootstrap.servers", "localhost:9092")
  props.put("client.id", "KafkaProducer")
  props.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer")
  props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer")

  val producer = new KafkaProducer[Integer, String](props)

  import org.apache.kafka.clients.producer.ProducerRecord

  val polish = java.time.format.DateTimeFormatter.ofPattern("dd.MM.yyyy H:mm:ss")
  val now = java.time.LocalDateTime.now().format(polish)
  val record = new ProducerRecord[Integer, String](topic, 1, s"hello at $now")
  val metaF: Future[RecordMetadata] = producer.send(record)
  val meta = metaF.get() // blocking!
  val msgLog =
    s"""
       |offset    = ${meta.offset()}
       |partition = ${meta.partition()}
       |topic     = ${meta.topic()}
     """.stripMargin
  println(msgLog)

  producer.close()
}
```

## Building Kafka from the sources

In order to run the client you should build Kafka from the sources first and publish the jars to the local Maven repository. The reason to have the build is that the producer uses the very latest Kafka Producer API.

Building Kafka from the sources is as simple as executing `gradle -PscalaVersion=2.11.7 clean releaseTarGz` in the directory where you `git clone https://github.com/apache/kafka.git`d [the Kafka repo from GitHub](https://github.com/apache/kafka.git).

    ➜  kafka git:(trunk) gradle -PscalaVersion=2.11.7 clean releaseTarGz
    Building project 'core' with Scala version 2.11.7
    ...
    BUILD SUCCESSFUL

    Total time: 1 mins 23.233 secs

I was building the distro against **Scala 2.11.7**.

Once done, `core/build/distributions/kafka_2.11-0.8.3-SNAPSHOT.tgz` is where you find the release package.

    ➜  kafka git:(trunk) ✗ ls -l core/build/distributions/kafka_2.11-0.8.3-SNAPSHOT.tgz
    -rw-r--r--  1 jacek  staff  17006303 18 lip 13:19 core/build/distributions/kafka_2.11-0.8.3-SNAPSHOT.tgz

Unpack it and `cd` to it.

    ➜  kafka git:(trunk) ✗ tar -zxf core/build/distributions/kafka_2.11-0.8.3-SNAPSHOT.tgz
    ➜  kafka git:(trunk) ✗ cd kafka_2.11-0.8.3-SNAPSHOT

## Zookeeper up and running

Running Zookeeper is the very first step you should do (as that's how Kafka maintains high-availability). Use `./bin/zookeeper-server-start.sh config/zookeeper.properties`:

    ➜  kafka_2.11-0.8.3-SNAPSHOT git:(trunk) ✗ ./bin/zookeeper-server-start.sh config/zookeeper.properties
    [2015-07-20 00:17:08,134] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
    [2015-07-20 00:17:08,136] INFO autopurge.snapRetainCount set to 3 (org.apache.zookeeper.server.DatadirCleanupManager)
    [2015-07-20 00:17:08,136] INFO autopurge.purgeInterval set to 0 (org.apache.zookeeper.server.DatadirCleanupManager)
    [2015-07-20 00:17:08,136] INFO Purge task is not scheduled. (org.apache.zookeeper.server.DatadirCleanupManager)
    [2015-07-20 00:17:08,136] WARN Either no config or no quorum defined in config, running  in standalone mode (org.apache.zookeeper.server.quorum.QuorumPeerMain)
    [2015-07-20 00:17:08,153] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
    [2015-07-20 00:17:08,154] INFO Starting server (org.apache.zookeeper.server.ZooKeeperServerMain)
    [2015-07-20 00:17:08,165] INFO Server environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT (org.apache.zookeeper.server.ZooKeeperServer)
    [2015-07-20 00:17:08,165] INFO Server environment:host.name=192.168.1.9 (org.apache.zookeeper.server.ZooKeeperServer)
    [2015-07-20 00:17:08,165] INFO Server environment:java.version=1.8.0_45 (org.apache.zookeeper.server.ZooKeeperServer)
    ...
    [2015-07-20 00:17:08,191] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)

## Kafka broker up and running

With Zookeeper up, start a Kafka broker using `./bin/kafka-server-start.sh config/server.properties` command:

    ➜  kafka_2.11-0.8.3-SNAPSHOT git:(trunk) ✗ ./bin/kafka-server-start.sh config/server.properties
    [2015-07-20 00:18:33,574] INFO KafkaConfig values:
    	advertised.host.name = null
      ...
    	log.dir = /tmp/kafka-logs
      ...
    	zookeeper.connect = localhost:2181
    	zookeeper.sync.time.ms = 2000
    	port = 9092
      ...
    [2015-07-20 00:18:33,671] INFO starting (kafka.server.KafkaServer)
    [2015-07-20 00:18:33,673] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
    [2015-07-20 00:18:33,684] INFO Starting ZkClient event thread. (org.I0Itec.zkclient.ZkEventThread)
    [2015-07-20 00:18:33,693] INFO Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT (org.apache.zookeeper.ZooKeeper)
    [2015-07-20 00:18:33,694] INFO Client environment:host.name=192.168.1.9 (org.apache.zookeeper.ZooKeeper)
    [2015-07-20 00:18:33,694] INFO Client environment:java.version=1.8.0_45 (org.apache.zookeeper.ZooKeeper)
    [2015-07-20 00:18:33,694] INFO Client environment:java.vendor=Oracle Corporation (org.apache.zookeeper.ZooKeeper)
    ...
    [2015-07-20 00:18:34,414] INFO Registered broker 0 at path /brokers/ids/0 with addresses: PLAINTEXT -> EndPoint(192.168.1.9,9092,PLAINTEXT) (kafka.utils.ZkUtils$)
    [2015-07-20 00:18:34,419] INFO [Kafka Server 0], started (kafka.server.KafkaServer)

## Create topic

You're now going to create `my-topic` topic where the custom producer is going to publish messages to. Of course, the name of the topic is arbitrary, but should match what the custom producer uses.

    ➜  kafka_2.11-0.8.3-SNAPSHOT git:(trunk) ✗ ./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic my-topic-test
    Created topic "my-topic-test".

Check out the topics available using `./bin/kafka-topics.sh --list --zookeeper localhost:2181`. You should see one.

    ➜  kafka_2.11-0.8.3-SNAPSHOT git:(trunk) ✗ ./bin/kafka-topics.sh --list --zookeeper localhost:2181
    my-topic-test

## Scala/sbt project setup

Create a Scala/sbt project with the following `build.sbt`:

    val kafkaVersion = "0.8.3-SNAPSHOT"

    scalaVersion := "2.11.7"

    resolvers += Resolver.mavenLocal

    libraryDependencies += "org.apache.kafka" % "kafka-clients" % kafkaVersion

Use the following `project/build.properties`:

    sbt.version=0.13.9-RC3

## Sending messages using KafkaProducer - `sbt run`

With the setup, you should now be able to run `sbt run` to run the custom Scala producer for Kafka.

    [kafka-publisher]> run
    [info] Running KafkaProducer
    Connecting to my-topic-test
    SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    SLF4J: Defaulting to no-operation (NOP) logger implementation
    SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

    offset    = 0
    partition = 0
    topic     = my-topic-test

    [success] Total time: 8 s, completed Jul 20, 2015 12:29:44 AM

Executing `sbt run` again should show a different offset for the sam partition and topic:

    [kafka-publisher]> run
    [info] Running KafkaProducer
    Connecting to my-topic-test
    SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
    SLF4J: Defaulting to no-operation (NOP) logger implementation
    SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

    offset    = 1
    partition = 0
    topic     = my-topic-test

    [success] Total time: 0 s, completed Jul 20, 2015 12:30:47 AM

## Using kafkacat as a Kafka message consumer

If you really would like to see the message on the other, receiving side, I strongly recommend using [kafkacat](https://github.com/edenhill/kafkacat) that, once installed, boils down to the following command:

    ➜  ~  kafkacat -C -b localhost:9092 -t my-topic-test
    hello at 20.07.2015 0:29:43
    hello at 20.07.2015 0:30:46

It will read all the messages already published to `my-topic-test` topic and print out others once they come.

That's it. Congratulations!

## Summary

The complete project is [on GitHub in kafka-producer repo](https://github.com/jaceklaskowski/kafka-producer).

You may also want to read [1.3 Quick Start](http://kafka.apache.org/documentation.html#quickstart) in the official documentation of Apache Kafka.

Let me know what you think about the blog post in the [Comments](#disqus_thread) section below or contact me at jacek@japila.pl.
