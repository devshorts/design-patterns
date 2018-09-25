---
title: Visitors
layout: page
permalink: /patterns/visitor
---

Lets talk about visitors

* Do not remove this line (it will not be displayed)
{:toc}

![](https://media.giphy.com/media/NDJWGU4n74di0/giphy.gif)

Visitor pattern is one that people often times have a hard time grokking.  This entire pattern predicates on the fact that a thing called [dynamic dispatch](https://en.wikipedia.org/wiki/Dynamic_dispatch) exists.  Dynamic dispatch is the fact that overloaded methods are determined _at runtime_ based on the referred to argument type. 

Given that, suppose we have the following base and derived classes:

```scala
class Base()
class Derived() extends Base

object Runner {
  def method(base: Base) = println("base")
  def method(derived: Derived) = println("derived")
}
```

If we were to call the overloaded methods with the following types:

```scala
val derived: Base = new Derived()
val base: Base = new Base()

Runner.method(base)
Runner.method(derived) 
```

What do you expect to be printed out? Turns out its 

```code
base
base
```

This is because the dispatcher selects the method based on the referred type, which we explicitly defined as `Base`.  How do we get this to do what we want which is to call `derived` for runtime types of `derived` without changing the variable definition?

The answer here is double dispatch. We need the method overloading to see the referred type as `Derived` and not actually as `Base`.  What if we refactor things a bit to look like:


```scala

class Runner {
  def method(base: Base) = {
    println("base")
  }

  def method(derived: Derived) = {
    println("derived")
  }
}

class Base {
  def execute(runner: Runner) = {
    runner.method(this)
  }
}

class Derived extends Base {
  override def execute(runner: Runner) = {
    runner.method(this)
  }
}
```

And we do now:


 ```scala
val base: Base = new Base
val derived: Base = new Derived
val runner = new Runner

base.execute(runner)
derived.execute(runner)
 ```
 
This now prints out:

```code
base
deried
```

What's the difference? The difference is that in the first example the compiler determined the overloaded method to call because we explicity said that the types were of type `Base`. To the compiler there's no question about what the type is, it doesn't care that the runtime value of `Base` may be `Derived`. However, in the second example, we forced an _instance_ of `Derived` to call into the runner so that the overloaded method resolves to the `Derived` instance and not the `Base` instance!

In this scenario, we did a double dispatch. We first dispatched on the `Base` object calling `execute`, then in `execute` we _dispatched again_ back to runner which resolves the correct overloaded method.

Why is this useful?  Double Dispatch is used to invoke an overloaded method where the parameters vary among an inheritance hierarchy [1].  This is especially handy when traversing things like trees or other hierarchies.  Imagine we have a really crappy programming language that consists only of if statements, boolean expressions, and brackets:


```scala
sealed trait SyntaxNode
case class If(predicate: SyntaxNode, body: SyntaxNode) extends SyntaxNode
case class Block(statements: List[SyntaxNode]) extends SyntaxNode
case class BooleanExpression() extends SyntaxNode
```

And we want traverse over a tree of this language starting at a block statement and print out the structure of the language.  Later if the structure is OK, we'll want to actually run the language.  Given we want to iterate over the tree in two _separate_ ways, lets make sure the iteration is decoupled from the implementation of the iteration.  First lets define a visitor that defines the overloaded methods we want to handle:

```scala
trait SyntaxVisitor {
  def visit(ifNode: If)
  def visit(blocK: Block)
  def visit(boolean: BooleanExpression)
}
```

Lets also make sure that syntax nodes are forced to implement a visitor with another trait:

```scala
trait SyntaxVisitable {
  def accept(visitor: SyntaxVisitor)
}

sealed trait SyntaxNode extends SyntaxVisitable
```

Now all our nodes have to implement an `accept` method that takes a visitor. The methods will _all look the same_:

```scala
case class If(predicate: SyntaxNode, body: SyntaxNode) extends SyntaxNode {
 override def accept(visitor: SyntaxVisitor) = visitor.visit(this)
}

case class Block(statements: List[SyntaxNode]) extends SyntaxNode {
  override def accept(visitor: SyntaxVisitor) = visitor.visit(this)
}

case class BooleanExpression() extends SyntaxNode {
  override def accept(visitor: SyntaxVisitor) = visitor.visit(this)
}
```

Now, we can actually implement a visitor and apply it. Remember, the visitors job is to just be able to properly see the runtime type! It's just a way of forcing the method overloading to work the way you want it to

```scala
class SimpleVisitor extends SyntaxVisitor {
  override def visit(ifNode: If): Unit = {
    println("Got an if")
    ifNode.predicate.accept(this)
    ifNode.body.accept(this)
  }

  override def visit(blocK: Block): Unit = {
    println("Got a block")
    blocK.statements.foreach(_.accept(this))
  }

  override def visit(boolean: BooleanExpression): Unit = {
    println("Got a boolean expression")
  }
}
```

And if we run this:

```scala
val language = Block(List(
  If(BooleanExpression(), Block(List(
    BooleanExpression()
  )))
))

new SimpleVisitor().visit(language)
```

We get the result of:

```
Got a block
Got an if
Got a boolean expression
Got a block
Got a boolean expression
```

Ok, neat, but so what.  The real power in this pattern is that the tree (or inheritance heirarchy) doesn't care _WHO_ is iterating over it. Just that someone IS iterating over it. We can make a different visitor that maybe actually runs our language.  Or maybe we have one that lints that the syntax is correct.  This is actually a really common pattern with language design. You have visitors that parse a syntax tree, you have visitors that compile a syntax tree, and you may have visitors that help you debug and print a syntax tree.  This lets you decouple how the tree is walked from the actual tree!

Visitors are difficult to wrap your head around because you have item A calling item B which calls item A again, and we're tricking the runtime into executing the correct overloaded method due to the fact that overloaded methods are determined at runtime by the type of the argument.  

In languages like scala that support pattern matching you don't really need to do a visitor pattern since you can force pattern matching on exhaustive traits to do the same kind of thing:

```scala
def matchLang(node: SyntaxNode): Unit = {
  node match {
    case If(predicate, body) =>
      println("If!")
      matchLang(predicate)
      matchLang(body)
    case Block(statements) =>
      println("Block!")
      statements.foreach(matchLang)
    case BooleanExpression() =>
      println("Boolean!")
  }
}

matchLang(language)
```

But we're not really relying on dispatching here, under the hood this is akin to doing instanceOf checks on items.  On top of that, if we don't specifically close the interface tree with a sealed trait AND we don't enable warnings as errors, then it is possible for us to add new items to the node tree and forget to implement visitor overloads for it.  However, with a visitor pattern when we add new types we MUST implement the visit method which is a great compile time safety net.

That said, visitors have other problems.  The biggest one is that in order to _return a result_ they tend to mutate themselves. This is because if we wanted to be able to return a result, we have to tell the visitor it can return a result, and all the visito methods must also return that type. This means that we can't have generic visitors that do _anything_. That said, you can totally make your visitor return a type if you are sure the type will always be returned.




---

[1] [Double Dispatch is a Code Smell](https://lostechies.com/derekgreer/2010/04/19/double-dispatch-is-a-code-smell/)