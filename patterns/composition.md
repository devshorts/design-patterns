---
title: Composition 
layout: page
permalink: /patterns/composition
---

* Do not remove this line (it will not be displayed)
{:toc}

Lets talk about composition (over inheritance)

![](https://media.giphy.com/media/aXbSfDEuF8XFm/giphy.gif)

In object oriented programming inheritance is a big deal (pun intended).  It defines that objects have heirarchical relationships of an "is-a" type. For example a dog "is-a" animal.  So you can model dog by having an animal class:

```scala
class Animal()
```

and having Dog subclass Animal:

```scala
class Dog() extends Animal
```

In many cases this makes sense.  Having inheritance lets you leverage structural properties like covariance and contravariance and lets you define logical heirarchies.

Where inheritance falls flat is in code re-use.  A lot times you will see code in the wild like:

```scala
// defines DB queries
class Database()
class Animal() extends Database
class Dog extends Animal()
``` 

What you see here is that we've defined a dog to be an animal. That makes sense. But an animal is also a... database? 

![](https://media.giphy.com/media/xThuW4BaAA2f7nRvoc/giphy.gif)

Certainly that doesn't make sense to be able to pass an animal to code that expects a database.  But, often times you do see heirarchies like this because they look to solve code re-use issues. After all you want to DRY (don't repeat yourself).  This is especially abusive in languages that support mix-ins or multiple inheritance. 

What we really want here isn't an _is a_ relationship, we want a _has a_ relationship.  To get ourselves out of _is a_ and into _has a_ we can  _compose_ objects of other objects. We can refactor this code smell into a heirarchy that makes sense while still allowing us to re-use code:

```scala
// defines DB queries
class Database()
class Animal(db: Database) 
class Dog(db: Database) extends Animal(db)
``` 

Now its clear that a `Dog` _is a_ `Animal` and that `Animal` _has a_ a database.  Advantages to this are

- Well defined object heirarchies that actually model the problem domain
- Easily testable components independently (a database can be tested outside of dog/animal)
- Extensible in you can now add other components and not worry about inheritance conflicts
- Explicit dependency information

 
