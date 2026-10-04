# 03. Functions and Methods

## Defining a method with `def`

```scala
// Scala 2 and 3
def add(a: Int, b: Int): Int = {
  a + b
}

// one-liner
def add(a: Int, b: Int): Int = a + b
```

```scala
// Scala 3 indentation style
def add(a: Int, b: Int): Int =
  val total = a + b
  total
```

> **🐍 Python equivalent**
> ```python
> def add(a: int, b: int) -> int:
>     return a + b
> ```

Key differences:
- Parameter types are **required**.
- The return type is usually optional (inferred), but write it on public methods. It is required for recursive methods.
- The `=` matters: the body is an expression, and its last value is returned.

> **🔀 2 vs 3: procedure syntax**
> Scala 2.12 allowed `def log(msg: String) { println(msg) }` (no `=`, returns `Unit`).
> This was deprecated in 2.13 and **removed in Scala 3**. Always write `: Unit =`.

## Default and named arguments

```scala
def greet(name: String, greeting: String = "Hello"): String =
  s"$greeting, $name!"

greet("Reetik")                       // Hello, Reetik!
greet("Reetik", "Hi")                 // Hi, Reetik!
greet(greeting = "Hey", name = "Ana") // named args, any order
```

These work just like Python's.

## Variable arguments

```scala
def sum(nums: Int*): Int = nums.sum
sum(1, 2, 3)              // 6

val xs = List(1, 2, 3)
sum(xs: _*)               // Scala 2: spread a collection (Scala 3 still accepts it, with a warning)
sum(xs*)                  // Scala 3
```

> **🐍 Python equivalent:** `def sum_(*nums)` and `sum_(*xs)`.
> Scala has no `**kwargs`.

## Functions as values (lambdas)

```scala
val double = (x: Int) => x * 2
double(5)                         // 10

val add: (Int, Int) => Int = (a, b) => a + b
```

> **🐍 Python equivalent**
> ```python
> double = lambda x: x * 2
> ```
> Unlike Python's `lambda`, Scala lambdas can have multiple lines:
> ```scala
> val process = (x: Int) => {
>   val y = x * 2
>   y + 1
> }
> ```

## Closures

A **closure** is a function that uses a variable defined *outside* itself
(a "free variable"). The function plus the variables it captured form the closure.

```scala
var multiplier = 3
val multiply = (x: Int) => x * multiplier   // captures `multiplier`

println(multiply(5))   // 15

multiplier = 10
println(multiply(5))   // 50  ← sees the updated value
```

Scala captures the **variable itself, not a copy of its value**, so later changes to a
captured `var` show up inside the function.

> **🐍 Python equivalent**
> Python closures behave the same way (late binding):
> ```python
> multiplier = 3
> multiply = lambda x: x * multiplier
> multiplier = 10
> multiply(5)   # 50
> ```

> **⚠️ Gotcha**
> Capturing a `var` makes the function's result depend on hidden mutable state.
> Capture `val`s whenever you can, so the closure always gives the same answer.

A common use is a function that **returns** a configured function:

```scala
def makeMultiplier(factor: Int): Int => Int =
  (x: Int) => x * factor        // `factor` is captured

val triple = makeMultiplier(3)
triple(5)                       // 15
```

> **🎯 Interview one-liner**
> "A closure is a function that references variables from its enclosing scope.
> It captures the variable itself, not a copy, so changes to a captured `var`
> are visible inside the function."

## Passing functions to functions

```scala
def applyTwice(f: Int => Int, x: Int): Int = f(f(x))

applyTwice(double, 3)        // 12
applyTwice(x => x + 10, 3)   // 23
applyTwice(_ + 10, 3)        // 23, using the placeholder syntax
```

`_ + 10` is shorthand for `x => x + 10`. Each `_` stands for the next parameter:

```scala
List(1, 2, 3).map(_ * 2)          // List(2, 4, 6)
List(1, 2, 3).reduce(_ + _)       // 6
```

## Methods vs functions

- `def` creates a **method** (belongs to a class/object).
- `val f = (x: Int) => ...` creates a **function value** (an object you can pass around).

You can pass a method where a function is expected; Scala converts it automatically:

```scala
def triple(x: Int): Int = x * 3
List(1, 2).map(triple)   // works in both 2 and 3
```

> **🔀 2 vs 3**
> In Scala 2, `val t = triple` (no arguments) is an error; you write `val t = triple _`.
> Scala 3 converts automatically, so `val t = triple` works.

## Currying and partial application

**Currying** turns a function that takes several parameters at once into a chain of
functions that take one parameter each: `f(a, b)` becomes `f(a)(b)`.

```scala
// Normal: both parameters at once
def add(x: Int, y: Int): Int = x + y
add(20, 10)                       // 30

// Curried by hand: returns a function value
def add2(x: Int): Int => Int = (y: Int) => x + y
add2(20)(10)                      // 30

// Curried with multiple parameter lists (the usual Scala way)
def add3(x: Int)(y: Int): Int = x + y
add3(100)(200)                    // 300
```

**Partial application** means fixing some arguments now and getting back a new
function that waits for the rest:

```scala
val sum40 = add2(40)              // already a function: (y: Int) => 40 + y
sum40(50)                         // 90

val sum50 = add3(50) _            // Scala 2 and 3
val sum50b: Int => Int = add3(50) // Scala 2 and 3 (expected type given)
val sum50c = add3(50)             // Scala 3 only
sum50(400)                        // 450
```

> **🔀 2 vs 3**
> In Scala 2, `add3(50)` on its own is a compile error ("missing argument list").
> You need the trailing `_` or an explicit function type.
> Scala 3 does this conversion automatically, and warns that `add3(50) _` is outdated syntax.

> **🐍 Python equivalent:** `functools.partial(add, 50)`

| | Currying | Partial application |
|---|---|---|
| What it does | Turns `f(a, b)` into `f(a)(b)` | Fixes some arguments of a function |
| Result | A chain of one-argument functions | A new function with fewer parameters |
| Related? | Currying makes partial application easy | |

Currying shows up all over the standard library. `foldLeft` is curried, so the
starting value goes in the first list and the function in the second:

```scala
List(1, 2, 3).foldLeft(0)(_ + _)  // 6
```

> **🎯 Interview one-liner**
> "Currying transforms a multi-parameter function into a sequence of single-parameter
> functions. It enables partial application: fixing some arguments up front to create
> specialised, reusable functions."

Multiple parameter lists also let callers pass a block in braces, which reads like a built-in construct:

```scala
def repeat(n: Int)(body: => Unit): Unit =
  for (_ <- 1 to n) body

repeat(3) {
  println("hi")
}
```

## By-name parameters (`=> T`)

In `body: => Unit` above, the argument is **not evaluated** until it is used,
and it is re-evaluated each time. That is why `println` runs 3 times, not once.

```scala
def logIfDebug(debug: Boolean, msg: => String): Unit =
  if (debug) println(msg)

logIfDebug(false, expensiveReport())  // expensiveReport() never runs
```

> **🐍 Python equivalent:** passing a `lambda: expensive_report()` and calling it inside.

## Recursion

```scala
import scala.annotation.tailrec

@tailrec
def factorial(n: Int, acc: BigInt = 1): BigInt =
  if (n <= 1) acc else factorial(n - 1, acc * n)

factorial(50)
```

`@tailrec` asks the compiler to turn the recursion into a loop, and fails to compile if it can't.

> **⚠️ Gotcha:** Python has no tail-call optimisation and a recursion limit of ~1000.
> Scala with `@tailrec` can recurse millions of times safely.

## Nested functions

```scala
def outer(x: Int): Int = {
  def helper(y: Int): Int = y * 10
  helper(x) + 1
}
```

Same as nested `def` in Python.

## Practice

1. Write `def isEven(n: Int): Boolean` and use it with `List(1,2,3,4).filter(isEven)`.
2. Write a curried `def power(exp: Int)(base: Int): Int` and create `square` from it.
3. Write a tail-recursive `fib(n)` with `@tailrec`.

---
Previous: [02. Control flow](02-control-flow.md) · Next: [04. Strings](04-strings.md)
