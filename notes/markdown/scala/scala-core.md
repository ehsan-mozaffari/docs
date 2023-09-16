# Scala core notes

## Definitions

### Intersection types:

**Intersection types:** Emulated by the `with` keyword in Scala 2.x and `&` in Scala 3. The type `A & B` represents values that are **both** of the type `A` and of the type `B` at the same time. Intersection types can be useful to describe requirements  *structurally* with a flawless type inference.

The *members* of an intersection type `A & B` are all the members of `A` and all the members of `B`.

```scala
trait Resettable:
  def reset(): Unit

trait Growable[A]:
  def add(a: A): Unit

def f(x: Resettable & Growable[String]): Unit =
  x.reset()
  x.add("first")


```

In the method `f` in this example, the parameter `x` is required to be *both* a `Resettable` and a `Growable[String]`.


### Premature Evaluation

Accidentally evaluate a statement too early and might assign it to a value or put it into a data structure which could casue premature evaluation.
