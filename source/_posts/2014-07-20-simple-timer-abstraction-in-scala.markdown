---
layout: post
title: "Simple Timer Abstraction in Scala"
date: 2014-07-20 20:25
comments: true
categories: scala
---

I've been digging into [Scala](http://www.scala-lang.org/) a lot more recently and one of the things that I love is the support for first class functions with a syntax
that is less verbose than Java (I think it is even better than Java 8's lambdas). For example, I was working on some test code to 
measure latency of various network operations. This is not production code so I don't need anything as powerful as Coda Hale's 
[Metrics](http://metrics.codahale.com/) library (which I love for production), I just needed a way to quickly time stuff in code.

In Scala this is as simple creating a function like this:

```scala
  def time(execution: () => Unit) {
    val start = System.currentTimeMillis()
    execution()
    val time = System.currentTimeMillis() - start
    println("execution time: " + time + "ms")
  }
```

This is a [higher-order function](http://en.wikipedia.org/wiki/Higher-order_function) that will time the execution of whatever 
function passed to it. If you are coming from Java it should look pretty straight forward, the only major difference is the function 
argument declaration of `execution: () => Unit`; this declares the function argument `execution` as type `() => Unit`. In Scala
the type declaration comes after the variable name and the two are separated by a `:`. The type declaration in this case defines
a function that takes zero arguments and that returns nothing (`Unit` here is similar to `void` in Java).

Below are some examples of using this utility function:

```scala
scala> time(() => println("sum of range 0 to 100000000: " + (0 to 100000000).reduce((a, b) => a + b)))
sum of range 0 to 100000000: 987459712
execution time: 3246ms

scala> time(() => Thread.sleep(500))
execution time: 500ms
```

This is only possible thanks to the ability to treat functions as objects. While this won't revolutionize the way you code it is does allow
you to start removing the boiler plate code that tends to build up in Java.

This is only the tip of the iceberg in functional programming, if you want to learn more check out the free [Scala by Example](http://www.scala-lang.org/docu/files/ScalaByExample.pdf) book provided
by on the Scala website.