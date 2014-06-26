---
layout: post
title: "Calculate profit in Scala with foldLeft"
date: 2014-06-26 22:57:00 +0200
comments: true
categories: scala
sidebar: collapse
---

With a credit in a foreign currency one may want to hedge to offset the foreign currency getting stronger, and hence increasing the cost of the credit.

Say, you bought 589 CHF when it costed 3.4007 PLN and then 593 for 3.3704. How much would you profit when the price of selling CHF rose to 3.4107 PLN?

<!-- more -->

The question of how much profit you earned with a series of pairs `(quantity, price)` against a given CHF price can be calculated as follows:

    def calculateProfit(series: Seq[(Int, Double)], currentPrice: Double): Double =
      series.foldLeft(0.0) {
        case (acc, (qty, price)) => acc + (currentPrice - price) * qty
      }

    val qtyPriceSeries = Seq((589,3.4007),(593,3.3704))
    val currPrice = 3.4107

    scala> calculateProfit(qtyPriceSeries, currPrice)
    res0: Double = 29.787899999999745

It gives you ca 30 PLN.

It'd be nice to have a series with the date when a given pair was made, and then compare it with other means of gaining profits. A web app developed in [Play](http://www.playframework.com/) and deployed to [Heroku](https://www.heroku.com/) or [CloudBees](http://www.cloudbees.com/) might be of help, wouldn't it?