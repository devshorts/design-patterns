---
title: Factories
layout: page
permalink: /patterns/factory
---

* Do not remove this line (it will not be displayed)
{:toc}

Lets talk about factories

![](https://media.giphy.com/media/10aADbYxnJlc9q/giphy.gif)

Factories make objects.  The first thing someone new to factories tends to ask is

> why use a factory? isn't `new` a factory?

Yes, it is! It's probably the most "raw" factory you have. It's a way of creating objects.  But sometimes creating objects gets complicated. Or sometimes you want to hide away how something is made and expose a way to get "new" instances of something without having to know all the inner details of how to create it. 

Lets build a house:

```scala
trait Feature
case object Bed extends Feature
case object Dresser extends Feature
case object Sink extends Feature
case object Stove extends Feature
case object Toilet extends Feature
case object Tv extends Feature
case object Couch extends Feature

case class Dimensions(x: Int, y: Int)

case class Room(
  dimensions: Dimensions,
  features: List[Feature]
)
case class House(  
  rooms: List[Room]
)
```

We've built a pretty generic house structure. A house is some set of rooms where each room has some size and set of features.  We can build any house we want! But, it might be nice to provide some easy way to build out common rooms:

```scala
object HouseFactory {
  def small = Dimensions(10, 10)
  def large = Dimensions(100, 100)
  
  def bedroom(size: Dimensions = small): Room = {
    Room(size, features = List(
      Bed, Dresser
    ))
  }
  
  def kitchen(size: Dimensions = small): Room = {
    Room(size, features = List(
      Sink, Stove
    ))
  }
  
  def bathroom(size: Dimensions = small): Room = {
    Room(size, features = List(
      Sink, Toilet
    ))
  }
  
  def living(size: Dimensions = small): Room = {
    Room(size, features = List(
      Couch, Tv
    ))
  }
}
```

Now we have a simple way of creating general rooms in our house. Nothing is stopping us from making a house that is more complicated, but if we want to piece together a 1 bedroom apartament, or a shack in the woods, we can probably do that pretty easily now:

```scala
// no bathroom since we have an outhouse
val shack = House(rooms = List(
   HouseFactory.bedroom(),
   HouseFactory.kitchen()
))

val oneBedroom = House(rooms = List(
  Housefactory.bedroom(),
  HouseFactory.kitchen(),
  HouseFactory.bathroom(),
  HouseFactory.living()
))
```

The power of a factory comes in providing sane and semantic ways of creating complicated objects and being able to pass around that factory as an object itself! Imagine another level of abstraction so that we can have different house factories depending on the criteria we want:

```scala
trait HouseFactory {
  def bedroom(size: Dimensions): Room
  def bathroom(size: Dimensions): Room
  def kithen(size: Dimensions): Room
  def living(size: Dimensions): Room
}
```

Lets make a fancy house factory with some extra features:

```scala
case object Butler extends Feature

class FancyHouseFactory extends HouseFactory {
  val huge = Dimensions(1000, 1000)
  
  // two beds, two sinks
  def bedroom(size: Dimensions = huge): Room = {
    Room(size, features = List(
      Bed, Bed, Dresser, Sink, Sink
    ))
  }
   
  def bathroom(size: Dimensions = huge): Room = {
    Room(size, features = List(
      Sink, Sink, Couch, Tv
    ))
  }
  
  def kithen(size: Dimensions = huge): Room = {
    Room(size, features = List(
        Sink, Sink, Stove, Stove, Tv
      ))
    }
  }
  
  def living(size: Dimensions = huge): Room = {
    Room(size, features = List(
        Butler, Tv, Couch, Couch
      ))
    }
  }
}
```

Oh man, faaaancy.  We can now pass around a `HouseFactory` trait and they'll all get fancy houses now.  If we want to make a `CrappyHouseFactory` we could do that too.  Once we have all these factories, we can apply things like [Strategies]({{ site.baseurl }}{% link patterns/strategy.md %}) to choose with factory to use.  