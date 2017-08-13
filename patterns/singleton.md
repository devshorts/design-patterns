---
title: Singletons
layout: page
permalink: /patterns/singleton
---

* Do not remove this line (it will not be displayed)
{:toc}

Lets talk about singletons

![](https://media.giphy.com/media/8SXE0qevEeb0A/giphy.gif)

THERE CAN BE ONLY ONE!!! Ok enough highlander stuff.

Singletons are where there is only ever _one_ instance of the object in your application.  Singletons can be static classes, or lazy instantiated classes that are only ever done once.  
Good usages for singletons are to encapsulate heavy resources. Some examples may be database abstractions since they tend to have connection pools, or caches because caches are only effective if they're actually the same instance, etc.

For all the good that singletons give, they get a lot of bad press. This is for a few reasons. 

## Problem 1: Hidden resource problem

One is that when they are used incorrectly they tend to hide that they are being used.  Here's what I mean:

```scala
// a database singleton
object Database {
  def read()
  def write()
}

class User {
   def getName() = {
      Database.read()
   }
}

class Main {
  new User().getName()
}
```

There's a few problems here:

1. Looking at just the signature of `User` you don't _know_ that it uses a database
2. Even if you did know it used a database, how do you test this without providing an actual database? Or a local in memory database? Or a mock database? 

Scala makes this a little simpler for us in that we can pass singletons (static classes, aka companion objects) around as instances, but other languages like Java/C#/etc do not.  In those languages you tend to "reach in" to resources directly without having them passed to you.

Another way of providing a singleton is to instantiate only one class ever:

```scala
object DatabaseProvider {
  // the only database we'll make
  lazy val database = new Database
}
// a database class
class Database {
  
  def read()
  def write()
}

class User(database: Database) {
   def getName() = {
      database.read()
   }
}

class Main {
  new User(DatabaseProvider.database).getName()
}
```

In this example we're passing a database to the user class. The user class doesn't _know_ or _care_ that the database is effectively a singleton. All it cares is that it got a database! Whether we instantiate the database in the companion object, or we provide it with a dependency injection framework (like spring/guice/whatever), or we create one instance of it in `Main` and pass it around doesn't matter anymore. It's still effectively the same instance floating around.

Making singletons by hand without the functionality of keywords like `lazy` in scala can be tricky, especially if you are lazily making singletons in a threaded environment.  For those interested in reading more about that they should read up on [singleton double checked locking](https://en.wikipedia.org/wiki/Double-checked_locking) 


## Problem 2: Thread safety

While that fixes the hidden resource problem, singletons still exhibit other potential land-mines.  When you have only one instance of an object in your codebase you need to be very aware of the concurrent usages of your class. If you are in a multi-threaded environment then your singleton _must_ be threadsafe. Without thread safety you risk corruption, especially if your singleton contains any kind state (like a cache).  

In general, it's best to avoid singletons for everything but when they are actually needed. In most cases objects are cheap to instantiate and cheap to collect (most objects are very short lived) so don't be afraid to make new instances of something when you need to.
