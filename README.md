# Async without Future


FTypes:

* right now, IS EXPERIMENTAL
* is a type system
* is for concurrent programming (async)
* solve the same problem than futures do (without futures)
* allows working with asynchronous types as if they were normal synchronous ones

```scala

import scala.concurrent.Future
import com.bryghts.ftypes._

val a = async.Int(Future.successful(3))
val b = async.Int(Future.successful(4))

val c: async.Int = a + b

```

# Getting Started

Include in your SBT file:

```sbt

addCompilerPlugin("org.scalamacros" % "paradise" % "2.1.0" cross CrossVersion.full)

libraryDependencies += "com.bryghts.ftypes" %% "ftypes" % "0.0.3"

```

Or, if you are using ScalaJS:

```sbt

addCompilerPlugin("org.scalamacros" % "paradise" % "2.1.0" cross CrossVersion.full)

libraryDependencies += "com.bryghts.ftypes" %%% "ftypes" % "0.0.3"

```

NOTE: The first line is to include the [Macro Paradise compiler plugin](http://docs.scala-lang.org/overviews/macros/paradise.html) which powers the [@Async](#case-classes) macro-annotation

# Sample
Imagine we have a function like this:

```scala

def getActiveServers(regionId: Int): Future[Int] = ???

```

And we want to compute how many servers we have if we combine Europe and America.

With standard Futures you would do something like this:

```scala

val europeCount:  Future[Int]  = getActiveServers(europeId)
val americaCount: Future[Int]  = getActiveServers(americaId)

val totalCount: Future[Int] =
    europeCount.flatMap(e => americaCount.map(a => e + a))

```

Or, to make it a bit nicer:

```scala

val totalCount: Future[Int] = for {
  e   <-   europeCount
  a   <-   americaCount
  } yield a + e

```

With FTypes would be something like this:

```scala

import com.bryghts.ftypes._

def getActiveServers(regionId: Int): async.Int = ???

val europeCount:  async.Int = getActiveServers(europeId)
val americaCount: async.Int = getActiveServers(americaId)

val totalCount:   async.Int = europeCount + americaCount

```

**async.Int** is just one of the types that FTypes provides out of the box, with practically all the same operations than the standard Int, where, like in the example, the '+' operation returns an async.Int that will hold the value of the sum (whenever the other two numbers are available)

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
 
