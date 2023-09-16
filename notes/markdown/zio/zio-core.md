# ZIO core notes

ZIO (Functional Effect Type) is a new library for concurrent programming. A functional effect is a kind of blueprint for a concurrent workflow. Using features of the Scala programming language, ZIO helps you build efficient, resilient, and concurrent
applications that are easy to understand and test, and which dont leak resources, deadlock, or lose errors.

Used pervasively across an application, ZIO simplifies many of the challenges of building modern applications:

* **Concurrency**. Using an asynchronous fiber-based model of concurrency that never blocks threads or deadlocks, ZIO can run thousands or millions
  of virtual threads concurrently.
* **Efficiency**. ZIO automatically cancels running computations when the result of the computations are no longer necessary, providing global application efficiency for free.
* **Error Handling**. ZIO lets you track errors statically, so the compiler can tell you which code has handled its errors, and which code can fail, including how it can fail.
* **Resource-Safety**. ZIO automatically manages the lifetime of resources, safely acquiring them and releasing them even in the presence of concurrency
  and unexpected errors.
* **Streaming**. ZIO has powerful, efficient, and concurrent streaming that works with any source of data, whether structured or unstructured, and never leaks resources.
* **Troubleshooting**. ZIO captures all errors, including parallel and finalization errors, with detailed execution traces and suspension details that make troubleshooting applications easy.
* **Testability**. With dependency inference, ZIO makes it easy to code to interfaces, and ships with testable clocks, consoles, and other core system modules.
* **Wonderful Community**. I'm amazed at all the talent, positivity, and mentorship seen in the ZIO community, as well as the universal attitude that we are all on the same team - there is no your code, or my code, just our code.
* **History of Innovation**. ZIO has been the first effect type with proper thread pool locking, typed errors, environment, execution tracing, finegrained
  interruption, structured concurrency, and so much more. Although inspired by Haskell, ZIO has charted its own course and become what many believe to be the leading effect system in Scala.
* **A Bright Future**. ZIO took three years to get right, because I believed the foundations had to be extremely robust to support a compelling nextgeneration ecosystem. If early libraries like Caliban, ZIO Redis, ZIO Config, ZIO gRPC, and others are any indication, ZIO will continue to become a hotbed of exciting new developments for the Scala programming language.

## Definitions

### Refrential transparency

An expression such as 2 + 2 is referentially transparent if we can always replace the computation with its result in any program while still preserving its runtime behavior.

### Side effect

Run things on the side.

### Pure functions

Expressions without side-effects are called p**ure expressions** while functions whose body are pure expressions are called **pure functions**.

### ZIO Environment:

* Todo: complete the definition

In `ZIO[R, E, A]`: Imagine ZIO is a function: `R => Either[E, A]`.

`ZIO[R, E, A]` type parameters:

* `R`: called the environment of the effect to be executed like database or might not require any environment and the paramerter will be `Any` meaning it could run on any environment or the environment not required! is contravariant
* `E`: is the type of value that the effect can fail with. This could be `Throwable `or `Exception`, but it could also be a *domain-specific error type*, or an
  effect might not be able to fail at all, in which case the type parameter will be `Nothing`.
* `A` is the type of value that the effect can succeed with. It can be thought of as the return value or output of the effect. Is covariant.

Using flawless [intersection types](../scala/scala-core.md) it could add more power to `ZIO` type as improves testability and type inference. It has an ability to describe dependencies using types without actually passing them.


## ZIO Type Aliases

```scala
type IO[+E, +A]   = ZIO[Any, E, A]         // No environment requirement
type Task[+A]     = ZIO[Any, Throwable, A] // No environment and failed with Throwable
type RIO[-R, +A]  = ZIO[R, Throwable, A]   // Failed with Throwable
type UIO[+A]      = ZIO[Any, Nothing, A]   // No environment and cannot fail
type URIO[-R, +A] = ZIO[R, Nothing, A]     // cannot fail
```


### ZIO STM: Software Transactional Memory

Inspired by Haskell data type.

ZIO STM features:

* Stack-safe for any transactions size
* Improve the preformance of existing structures
* Implement retry-storm protection for supporting large transactions on htly contested transactional references

You can build concurrent structure with the followings:

* `Promise`: a variable data type that can be set exactly one time (but can
  be awaited on asynchronously and retrieved any number of times)
* `Ref`: a model of a mutable cell that can store any immutable value, with
  atomic operations for updating the cell.

### ZIO Execution Traces

Is a tool for debug ZIO code. It dynamically parsing the bytecode of class files.

### ZIO Test

A purely functional testing framework. It is hard to test functional effects insde a traditional testing libraries.

Like `Specs2` ZIO test embraced a philosophy of tests as values. It is a fatherweight alternative to `ScalaCheck` based on ZIO Stream since the generator of a value can be viewed as a stream. IT inspired by Haskell Hedgehog library to has auto-shrinking baked in to handle filters on shrunk values and other edge scenarios that `ScalaCheck` did not handle.


### ZIO ZLayer

It was built because of the pain in constructing larger ZIO Environment with two new data types:

* `Has`: can be thought of as a type-indexed heterogeneous map, which is type safe,
  but requires access to compile-time type tag information.
* `ZLayer`: can be thought of as a more powerful version of Java and Scala constructors, which can build multiple services in terms of their dependencies.

Unlike constructors, ZLayer dependency graphs are ordinary values, built from other values using composable operators, and ZLayer supports resources, asyn-chronous creation and finalization, retrying, and other features not possible with constructors.


### ZIO Structured Concurrency

Structured concurrency is a paradigm that provides strong guarantees around the lifespans of operations performed concurrently. These guarantees make it easier to build applications that have stable, predictable resource utilization.

ZIO was the first effect system to support structured concurrency in numerous operations:

* By default, interrupting a fiber does not return until the fiber has been interrupted and all its finalizers executed.
* By default, timing out an effect does not return until the effect being timed out has been interrupted and all its finalizers executed.
* By default, when executing effects in parallel, if one of them fails, the parallel operation will not continue until all sibling effects have been
  interrupted.

### ZIO function description

* **ZIO.attemp**: Is a constructor blueprint to build ZIO effect that describes but doesn't actually do anything right now. converting exception in Left and success in Right values.
* **ZIO.delay**: Is another description to transform the effect into a new effect. For example: `ZIO(println("hello world!")).delay(1.hour)`
* **ZIO.fail**: shows that the effect fails eventually.
* **ZIO.succeed**: shows the effect succeeded eventually.
* **ZIO flatMap**: a fundamental operator represents sequential composition of two effects, allowing us to create a second effect based on the output of the first effect. Using this sequential operator, we can describe a simple workflow that reads user input and then displays the input back to the user. We can translate any procedural program into ZIO by wrapping each statement in a constructor like `ZIO.attempt` and then gluing the statements using `flatMap`. And to use it more elegantly you could use *for emprehensions* to look like procedural programming. A *for comprehension* with *n* lines is translated by Scala into *n - 1* calls to `flatMap` methods on the effects, followed by a final call to a map method on the last effect.
* `zipWith`: combines two effect sequentially and merging their tow results with a user-defined function. For example: `val fullName = firstName.zipWith(lastName)((first, last) => s"$first first$last")`
* `zipLeft`: sequentially combines the result of two effects returning the result of the first. The alias `<*`. This is useful while `Unit` is not a very useful success value use this operator.
* `zipRight`: sequentially combines the result of two effects returning the result of the second. The alias `*>`. This is useful while `Unit` is not a very useful success value use this operator.
* `ZIO.foreach`: `ZIO.foreach(1 to 100) { n => printLine(n.toString) }`
* `ZIO.collectAll`: `ZIO.collectAll(<List of effects>)`
* `ZIO.fromEither`
* `ZIO.fromOption`
* `ZIO.fromTry`
* `ZIO.async`: It is tricky to underestand,
