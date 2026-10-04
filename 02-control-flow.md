# 02. Control Flow as Expressions

The biggest mindset shift from Python: in Scala, **almost everything is an expression
that returns a value**. `if`, `match`, `try` and blocks `{ ... }` all produce values.

## Blocks return their last line

```scala
val total = {
  val a = 10
  val b = 20
  a + b          // last expression is the block's value
}
// total == 30
```

No `return` needed. Using `return` in Scala is legal but considered bad style.

## `if` / `else`

```scala
// Scala 2 and 3
val label = if (age >= 18) "adult" else "minor"
```

```scala
// Scala 3 alternative syntax (no parentheses, uses `then`)
val label = if age >= 18 then "adult" else "minor"
```

> **🐍 Python equivalent**
> ```python
> label = "adult" if age >= 18 else "minor"
> ```
> Scala's `if` *is* the ternary. There is no separate `? :` operator.

Multi-branch:

```scala
// Scala 2
val grade =
  if (score >= 90) "A"
  else if (score >= 75) "B"
  else "C"

// Scala 3
val grade =
  if score >= 90 then "A"
  else if score >= 75 then "B"
  else "C"
```

> **⚠️ Gotcha**
> An `if` without `else` returns `Unit`, so `val x = if (cond) 5` has type `AnyVal`, not `Int`.
> Always write the `else` when you use `if` as a value.

## `match`: Python's `match` on steroids

```scala
val day = 3
val name = day match {
  case 1 => "Monday"
  case 2 => "Tuesday"
  case 3 | 4 | 5 => "Midweek"
  case _ => "Weekend"      // _ is the wildcard, like Python's case _:
}
```

> **🐍 Python equivalent (3.10+)**
> ```python
> match day:
>     case 1: name = "Monday"
>     case 2: name = "Tuesday"
>     case 3 | 4 | 5: name = "Midweek"
>     case _: name = "Weekend"
> ```
> The Scala version returns a value directly.

With guards and types:

```scala
def describe(x: Any): String = x match {
  case 0                => "zero"
  case n: Int if n < 0  => s"negative int $n"
  case n: Int           => s"positive int $n"
  case s: String        => s"a string of length ${s.length}"
  case _                => "something else"
}
```

> **⚠️ Gotcha**
> If no case matches, Scala throws a `MatchError` at runtime. Add `case _` when unsure.

Pattern matching gets its own deep-dive in topic 13.

## `while` loops

```scala
var i = 0
while (i < 3) {          // Scala 3 also allows: while i < 3 do
  println(i)
  i += 1
}
```

`while` returns `Unit`. You will rarely need it; collection methods and `for` cover most cases.

> **🔀 2 vs 3**
> Scala 2.x had `do { ... } while (cond)`. Scala 3 **removed** it.
> Scala 3 also uses `do` in `while cond do ...` syntax instead.

There is **no `break` or `continue` keyword**. Use `scala.util.control.Breaks`
if you must, but usually a method like `find`, `takeWhile` or `exists` is cleaner.

## `for` loops

```scala
// Scala 2
for (i <- 1 to 3) println(i)          // 1, 2, 3   (inclusive)
for (i <- 1 until 3) println(i)       // 1, 2      (exclusive, like range(1, 3))

// Scala 3
for i <- 1 to 3 do println(i)
```

> **🐍 Python equivalent**
> ```python
> for i in range(1, 4): print(i)
> ```

Nested loops and filters in one `for`:

```scala
for {
  x <- 1 to 3
  y <- 1 to 3
  if x != y
} println(s"$x,$y")
```

## `for`/`yield`: comprehensions

Add `yield` and the `for` builds a new collection:

```scala
val squares = for (x <- List(1, 2, 3, 4) if x % 2 == 0) yield x * x
// List(4, 16)
```

> **🐍 Python equivalent**
> ```python
> squares = [x * x for x in [1, 2, 3, 4] if x % 2 == 0]
> ```

Under the hood, `for`/`yield` is translated to `map`, `flatMap` and `withFilter`.
That is why it also works with `Option`, `Future` and `Either` (topics 11, 12, 18).

## `try` / `catch` / `finally`

```scala
val n = try {
  "42x".toInt
} catch {
  case e: NumberFormatException => 0
} finally {
  println("done")
}
// n == 0
```

`try` is an expression too. Topic 12 shows the more idiomatic `Try` and `Either`.

## 🔀 2 vs 3 summary

| Construct | Scala 2 | Scala 3 (new style, optional) |
|-----------|---------|-------------------------------|
| if | `if (c) a else b` | `if c then a else b` |
| while | `while (c) { ... }` | `while c do ...` |
| for | `for (x <- xs) ...` | `for x <- xs do ...` |
| for/yield | `for (x <- xs) yield f(x)` | `for x <- xs yield f(x)` |
| braces | required | optional; indentation works like Python |

Scala 3 accepts the Scala 2 syntax too, so old code keeps compiling.

## Practice

1. Write FizzBuzz for 1 to 15 using `for` and `match`.
2. Rewrite `[x * 2 for x in range(10) if x % 3 == 0]` as a Scala `for`/`yield`.
3. What is the type of `if (true) 1 else "one"`? Check in the REPL.

---
Previous: [01. Values and types](01-values-and-types.md) · Next: [03. Functions](03-functions.md)
