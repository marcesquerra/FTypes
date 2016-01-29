# What is FTypes?


FTypes:

* Right now: **IS EXPERIMENTAL**
* Is a type system (that is: a collection of base types plus means for building complex types out of simpler ones)
* Is for concurrent programming (async)
* Solve the same problem than futures do. But without futures (or nearly so)
* Allows working with asynchronous types as if they were normal synchronous ones
* These async types have the same interface (methods) than their synchronous counterparts

```scala

import scala.concurrent.Future
import com.bryghts.ftypes._

val a = async.Int(Future.successful(3)) // Convert Future into async
val b = async.Int(Future.successful(4))

val c: async.Int = a + b

```

# Getting Started

Include in your SBT file:

```sbt

addCompilerPlugin("org.scalamacros" % "paradise" % "2.1.0" cross CrossVersion.full)

libraryDependencies += "com.bryghts.ftypes" %% "ftypes" % "0.0.3"

```

Or, if you are using [ScalaJS](http://www.scala-js.org/):

```sbt

addCompilerPlugin("org.scalamacros" % "paradise" % "2.1.0" cross CrossVersion.full)

libraryDependencies += "com.bryghts.ftypes" %%% "ftypes" % "0.0.3"

```

NOTE: The first line is to include the [Macro Paradise compiler plugin](http://docs.scala-lang.org/overviews/macros/paradise.html) which powers the [@Async](#case-classes) macro-annotation

# Example
Imagine we have coded a function that returns how many healthy and active servers we have, for a particular region. The response of such a method would quite possibly be asynchronous, which, if we want to access the value, usually opens up three possibilities:

* Blocking. Currently, in the Scalasphere this is considerd a **Very Bad Idea** (I'm looking on sources to support this point)
* Callbacks. Nearly as bad a blocking ([Google for "Callback Hell"](http://www.google.com/search?q=callback+hell))
* Futures (of which FTypes are a variant)

In this example, I'm going to compare standard scala Futures with FTypes.

## With Futures
In this case, our method would have a signature more or less like this:

```scala

import scala.concurrent._

def getActiveServers(regionId: Int): Future[Int] = ???

```

And, if we want to compute how many servers we have if we combine Europe and America:


```scala

val europeCount:  Future[Int]  = getActiveServers(europeId)
val americaCount: Future[Int]  = getActiveServers(americaId)

val totalCount: Future[Int] =
    europeCount.flatMap(e => americaCount.map(a => e + a))

```

Or, with for comprehencion, we can make it a bit nicer:

```scala

val europeCount:  Future[Int]  = getActiveServers(europeId)
val americaCount: Future[Int]  = getActiveServers(americaId)

val totalCount: Future[Int] = for {
  e   <-   europeCount
  a   <-   americaCount
  } yield a + e

```

## With FTypes
Here, our method would have a really similar signature:


```scala

import com.bryghts.ftypes._

def getActiveServers(regionId: Int): async.Int = ???

```

The real difference comes when we use this method to the same computation than before:

```scala

val europeCount:  async.Int = getActiveServers(europeId)
val americaCount: async.Int = getActiveServers(americaId)

val totalCount:   async.Int = europeCount + americaCount

```

**async.Int** is just one of the types that FTypes provides out of the box, with, practically, all the same operations (methods) than the standard Int, where, like in the example, the '+' operation returns an async.Int that will hold the value of the sum (whenever the other two numbers are available)


# Futures without Future
Most Scala devs have to work with [Futures](http://docs.scala-lang.org/overviews/core/futures.html) in a daily basis and know the toll they apply in the mental model and the code readability/maintainability. If you don't know what a Future is, you can find an excellent introduction in [this article](http://danielwestheide.com/blog/2013/01/09/the-neophytes-guide-to-scala-part-8-welcome-to-the-future.html) by Daniel Westheide.

If you know what a future is and how it works, let me present you with a comparison. On one side, we have:

```scala
Future[Int]
```

On the other hand, we have:

```scala
async.Int
```

Both represent exactly the same thing, an Int value that will be available at some, unknown, point in time.

Comparing both snippets, you can draw an immediate conclusion. While a Future is a generic type (a Future can hold any type of data), the `async.Int` is a concrete type that con only hold integers. This means that if you want some sort of asynchronous `Boolean`, with Futures you have the problem already solved (i.e. `Future[Boolean]`) while with FTypes you need another specific type (which, by the way, FTypes already provides and is called `async.Boolean`).

The **main difference** comes when you compare the methods provided by `Future` and `async.Int`. Future is a generic type and knows nothing about the data is going to end up holding, were async.Int is always going to be used to only hold ints. In fact `async.Int` has none of the methods provided by the `Future` class but (nearly) all the methods provided by a regular, synchronous `Int`.

That is: With an `async.Int` you operate exactly the same as if it was a normal `scala.Int`

And this has a nice consequence: Functions that before received regular, synchronous types:

```scala
def average(a: Double, b: Double): Double = (a + b) / 2
```

Can now be reimplemented for the asynchronous world like this:

```scala
def average(a: async.Double, b: async.Double): async.Double = (a + b) / 2
```


# Interoperativility

Eventually (probably in the extremes of your code) you will need to operate with a more standard Scala way. FTypes provides an easy way to convert to (and from) Futures (Futures to the underlaying synchronous type):

```scala

val fi: Future[Int] = ???
val i = async.Int(fi)


val b: async.Boolean = ???
val fb: Future[Boolean] = b.future

```

# Base types

For now, the following base types are implemented:

- async.Byte
- async.Char
- async.Short
- async.Int
- async.Long
- async.Float
- async.Double
- async.Boolean
- async.String (this is a work in progress)

# Arrays

You can create immutable async arrays like with basic arrays:

```scala

val a = async.Array[async.Int](1, 2, 3)
val b = a(1)
val l = l.length  // returns an async.Int

```

NOTE: async arrays can only hold async types

# Case classes

This bit is extremely experimental, but thanks to the power of macros (and macro annotations), creating async case classes is as simple as this:

```scala

@Async case class Person(name: async.String, age: async.Int)

```

NOTE: as in the case of Array, async case classes can only hold async fields

# Compatibility

The project is provided for:
- JVM
- ScalaJS
 
And Scala Versions:
- 2.10
- 2.11


# Roadmap

- Improve this documentation (including the SBT section)
- Finish the String implementation
- Create async versions of StringOps, RichInt, ...
- Create an async version of the collections library
- Create the Option and Try monads
 

# Layers

This project don't contain much code by itself. It's instead a composition of the following projects:
- [Vals](https://github.com/marcesquerra/FTypes-Vals) Core system and base types (childs of "AnyVal")
- [Array](https://github.com/marcesquerra/FTypes-Array)
- [String](https://github.com/marcesquerra/FTypes-String)
- [CaseClass](https://github.com/marcesquerra/FTypes-CaseClass) Provides the "@Async" annotation for case classes. Require Macro Paradise
 
