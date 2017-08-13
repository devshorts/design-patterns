---
title: Strategies
layout: page
permalink: /patterns/strategy
---

* Do not remove this line (it will not be displayed)
{:toc}

Lets talk about strategies

![](https://media.giphy.com/media/wF8m2noseidDa/giphy.gif)

The strategy pattern is just a fancy way of saying of providing several implementations to an interface and choosing which implementation to use based on some criteria. To quote wikipedia:

> It  defines a family of algorithms, encapsulates each algorithm, and makes the algorithms interchangeable within that family.

A great example is retrying. Retries are common in software. You have transient errors all the time: network connectivity issues, file system locking, timeouts, crashes, etc.  Building robust software requires generous use of retries.. up to a point.  You don't want to retry forever and depending on what you are retrying you may want to retry a few times really quick and then give up, or maybe a couple times really quickly and then slower until a maximum amount of retries occurs, or maybe fast really quick then slow for a while and then peak out and try forever.

Each of those options is a strategy for how to retry.  We can implement a strategy pattern pretty easily and lets do an example with retries.

```scala
trait RetryStrategy {
  def execute[T](block: => T): T
}
```

This defines a strategy that given some block of code `block` that returns some value of type `T` we'll try to do it. If it fails, the strategy will retry until it either succeeds or the strategy gives up and throws an error.

Lets define a family of implementations:

```scala
case class ConsistentWaitRetry(
  maxTimes: Int, 
  waitBetweenFailuresMilli: Int
) extends RetryStrategy { ... }

case object NoRetry extends RetryStrategy { ... }

case class ExponentialRetry(
  maxTimes: Int, 
  waitBetweenFailuresMilli: Int,
  maxWaitTimeMilli: Int
) extends RetryStrategy { ... }
```

You can see we now can choose between 1 of these 3 stratgies whenever we want to do a retry.  Anyone who needs to retry can accept the `RetryStrategy` trait and it shouldn't care what the strategy _actually_ is, just that its contract says it will retry in some fashion.

To drive the point home lets implement a simple consistent wait retry that retries every N milliseconds up to Y times

```scala
case class ConsistentWaitRetry(
  maxTimes: Int, 
  waitBetweenFailuresMilli: Int
) extends RetryStrategy {
  override def execute[T](block: => T): T = {
     var timesToRetry = maxTimes
     while(timesToRetry > 0) {
        Try(block) match {
          case Failure(ex) =>
            timesToRetry -= 1 
            Thread.sleep(waitBetweenFailuresMilli)
          case Success(value) =>
            return value
        } 
     }
     
     throw new RetryFailedException()
  }
}
```

Now we can use it like:

```scala
val strategy: Strategy = ConsistentWaitRetry(
  maxTimes = 10, 
  waitBetweenFailuresMilli = 10
)

strategy.execute {
  throw new RuntimeException("I am always failing!")
}
```

And this will retry the block 10 times, waiting 10 milliseconds between each retry until it fails with a hard error.

If we want to make other strategies, like `ExponentialRetry` or `LinearRetry` we can do that.  The strategy part of this pattern is _choosing_ which implementation to use given some criteria.  Each implementation is a strategy, so however you decide to choose a strategy they are interchangeable due to adhering to a common contract.  You can even do things now like:

```scala
class Worker(retryStrategy: RetryStrategy) {
  def doWork(): Unit = {
    retryStrategy.execute {
      // some work
    }
  }
}

class WorkerUser(name: String) {
  def doWork: Unit = {
    val strategy: RetryStrategy = name match {
      case "Anton" => 
        ConsistentWaitRetry(...)
      case "Joel" =>
        ExponentialRetry(...)
      case _ =>
        NoRetryStrategy()
    }
    
    new Worker(strategy).doWork()
  }
}
```

Assuming we've built out the corresponding strategies (and we can even build out our own custom ones!) we can now choose, given the criteria of the worker name, which strategy to use. The worker itself doesn't _care_ what the strategy is, just that it was given one.

This makes the strategy pattern really flexible in choosing how to execute something at runtime.

I often times ask an interview question for candindates to design pacman. One thing people struggle with is abstracting how the ghosts who chase pacman _find_ pacman. This is a great way to provide strategies. Some examples of how a ghost can find pacman include

- Naively (randomly moving around)
- A* (tracking pacman and trying to find the least distance to them)
- AI (predict pacmans movements based on past movements)

If you abstract how the ghosts look for pacman, you can build a ghost class that is _pluggable_ with how it finds pacman. This means instead of having 10 ghost (sub)classes that each handle how movement occurs, you can have 1 class who is given a strategy.  This also is a great example of [composition]({{ site.baseurl }}{% link patterns/composition.md %}).

