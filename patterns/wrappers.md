---
title: Wrappers
layout: page
permalink: /patterns/wrappers
---

Lets talk about wrappers

* Do not remove this line (it will not be displayed)
{:toc}

![](https://media.giphy.com/media/l0MYHwUkOVWsTGErS/giphy.gif)

Wrappers usually wrap an object
and provide either extra functionality explicity (exposing new methods) or implicity (providing the same contract as the wrapped object).  Lets look at some examples. Wrappers come in many names: adapter, decorator, proxy, but they all tend to do the same thing. They wrap something.

Personally I find that this is maybe _the most_ useful pattern out there. It's so common I hardly ever think about the names of the patterns anymore.  I should note that just because you _use_ a pattern doesn't necessitate the name of the pattern being in the class/interface name. 

# Proxy

A proxy is a wrapper that takes a thing but gives the contract of the underlying object.

Lets pretend we have a state trait that defines how to get and put something based on a key.

```scala
trait State {
  def save[T](key: String, item: T): Unit
  def get[T](key: String): Option[T]
  def delete(key: State)
}
```

We can have a simple implementation of our state:

```scala
class InMemoryState extends State {
  private val map = new mutable.HashMap[String, Any]
  
  override def save[T](key: String, item: T): Unit = {
     map.add(key, item)
  }
  
  override def get[T](key: String): Option[T] = {
     map.get(item).map(_.asInstanceOf[T]))
  }
  
  override def delete(key: String): Unit = {
    map.remove(key)
  }
}
```

This works great.  But what if we want to now add monitoring to our state object.  We can obviously put monitoring into our state object:

```scala
class InMemoryState extends State {
  private val monitor: Monitoring =  ...
  
  private val map = new mutable.HashMap[String, Any]
  
  override def save[T](key: String, item: T): Unit = {
     monitor.increment("save")
     
     map.add(key, item)
  }
  
  override def get[T](key: String): Option[T] = {
     monitor.increment("get")
  
     map.get(item).map(_.asInstanceOf[T]))
  }
  
  override def remove(key: String): Unit = {
    monitor.increment("remove")
    
    map.remove(key)
  }
}
```

But the problem here is for every state implementation we create, we need to make sure we consistently add monitoring to it.  Especially problematic is if the state implementation is not owned by us. Maybe this `InMemoryState` comes from a 3rd party library. How do we add monitoring to it?  Enter the proxy.

The proxy will take an instance of `State`, but also implement `State` and _proxy_ calls to the
inner object.  For each call it will add monitoring itself:

```scala
class MonitoringStateProxy(state: State) extends State {
  private val monitor: Monitoring =  ...
       
  override def save[T](key: String, item: T): Unit = {
     monitor.increment("save")
     
     state.save(key, item)
  }
  
  override def get[T](key: String): Option[T] = {
     monitor.increment("get")
  
     state.get[T](item)
  }
  
  override def remove(key: String): Unit = {
    monitor.increment("remove")
      
    state.remove(key)
  }
}
```

What we have now is a proxy.  Anyone who accepts an instance of `State` won't know, or _care_ that the state they got isn't the one they used to use.  We can now easily add monitoring to any state implementation!  

This kind of stuff is especially useful for things like

- Extra failure handling or fallback functionality on failure
- Consistent monitoring or logging
- Controlling access to resources (via permissions/etc)
- Adding caching to heavy resources

The big win is the fact that the proxy implements the same trait as the underlying resource. This forces the proxy to handle all methods of the trait so if the trait changes, the proxy has to follow suit.  

# Decorator

A decorator is like a proxy++.  It usually (but not always) implements the source interface but also may implement other interfaces OR just expose extra methods itself.

Lets make a new state object, but this time lets add some extra methods that let us fix keys based on a prefix:

```scala
class FixedState(state: State) extends State { 
  override def save[T](key: String, item: T): Unit = {            
     state.save(key, item)
  }
  
  override def get[T](key: String): Option[T] = {    
     state.get[T](item)
  }
  
  override def remove(key: String): Unit = {   
    state.remove(key)
  }
  
  def saveWithPrefix(prefix: String, key: String, item: T): Unit = {
    save[T](prefix + key, item)
  }
}
```

Notice the new `saveWithPrefix` method. What we've done is taken an object of type `State` and "decorated" it with new functionality.  The scala world does this all the time with `Rich` methods (also known as the `Pimping` pattern).  If we were to rewrite this with the Rich extension method pattern, we'd do it like:

```scala
object StateExtensions {
  implicit class FixedState(state: State) {    
    def saveWithPrefix(prefix: String, key: String, item: T): Unit = {
      state.save[T](prefix + key, item)
    }
  }
}
```

To use this, we'd do:

```scala
// this provides the implicit conversion to a `FixedState` object
import StateExtensions._ 

val state: State = ...

state.saveWithPrefix(prefix = "foo", key = "bar", value = 123)
```
This effectively acts as a decorator pattern even though it doesn't actually implement the same interface.  

# Adapters

Adapters are a way to square peg round hole something.  Imagine you have two inconsistent interfaces

```scala
trait Dog {
  // returns a byte stream of a bark sound
  def bark(): AudioStream
}

trait Cat {
  // returns a byte stream of a cat sound
  def meow(): AudioStream
}
```

And you have some code that takes a `Dog`.  But lets say you only have a `Cat` laying around. Is it possible to transform a cat into a dog in such a way that the code that wants the dog can take a cat?  Maybe...

```scala
class CatLikeDog(cat: Cat) extends Dog {
  def bark(): AudioStream = {  
    cat.meow()
  }
}
```

Ok, I will admit this isn't a great example, a cat isn't a dog. But... it can pass as a dog (albeit a weirdly meowing one)! The idea behind an adapter is to do provide the adaptation mechanism to allow an object to behave like something it's not. In a non trivial example the adapter may take several other classes and utilities that help it achieve its goal.  