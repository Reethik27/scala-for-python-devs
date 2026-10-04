# 15. Option: Values That Might Be Missing

`Option` is how Scala handles "maybe there's a value, maybe there isn't" without
`null`. An `Option[A]` is one of two things:

```scala
Some(value)   // the value exists and is wrapped inside
None          // there is no value
```

> **🐍 Python equivalent**
> Python uses `None` directly, and type hints say `Optional[int]` / `int | None`.
> The difference: Python lets you forget to check for `None` and crash later with
> `AttributeError: 'NoneType' object has no attribute ...`. Scala's compiler won't let
> you use an `Option[Int]` as an `Int` until you deal with the `None` case.

## Where Options come from

The standard library returns `Option` whenever a lookup might fail:

```scala
val lst = List(1, 2, 3)
val map = Map(1 -> "Tom", 2 -> "Max", 3 -> "John")

lst.find(_ > 2)        // Some(3)
lst.find(_ > 100)      // None
lst.headOption         // Some(1)
List[Int]().headOption // None
map.get(1)             // Some(Tom)
map.get(99)            // None
"42".toIntOption       // Some(42)   (2.13+ / Scala 3)
```

Compare with the crashing versions:

```scala
map(99)               // NoSuchElementException: key not found
List[Int]().head      // NoSuchElementException: head of empty list
```

## Declaring Options

```scala
val a: Option[Int]    = Some(55)
val b: Option[Int]    = None
val c: Option[String] = Some("Hello")
val d = Option(null)  // None, Option(...) turns null into None (useful with Java APIs)
```

## Checking and extracting

```scala
a.isDefined           // true   (has a value)
b.isDefined           // false
a.isEmpty             // false
b.isEmpty             // true

a.get                 // 55
// b.get              // NoSuchElementException ← avoid .get!

a.getOrElse(0)        // 55
b.getOrElse(0)        // 0 (your default)
b.getOrElse(-1)       // -1

b.orElse(Some(99))    // Some(99), fall back to another Option
a.contains(55)        // true
```

> **⚠️ Gotcha: avoid `.get`**
> `.get` on a `None` crashes, which puts you right back where `null` was.
> Use `getOrElse`, `match`, `map` or `fold` instead.

> **⚠️ Gotcha: `getOrElse` with a different type**
> `b.getOrElse("Not found")` compiles, but the result type is `Any`, because it's
> either an `Int` or a `String`. Keep the default the same type as the value, or
> convert first: `b.map(_.toString).getOrElse("Not found")`.

## Pattern matching

The clearest way to handle both cases:

```scala
def describe(opt: Option[Int]): String = opt match {
  case Some(n) if n > 50 => s"big number $n"
  case Some(n)           => s"number $n"
  case None              => "nothing"
}

describe(Some(55))   // big number 55
describe(None)       // nothing
```

## Working with the value inside (without unwrapping)

Think of an `Option` as a collection with 0 or 1 elements. `map`, `filter`,
`flatMap` and `foreach` all work, and do nothing on `None`:

```scala
val opt: Option[Int] = Some(55)

opt.map(_ * 2)               // Some(110)
None.map((x: Int) => x * 2)  // None
opt.filter(_ > 100)          // None (value didn't pass)
opt.foreach(println)         // prints 55; prints nothing for None
opt.fold("empty")(n => s"got $n")  // "got 55"; "empty" for None
opt.exists(_ > 50)           // true
opt.toList                   // List(55)
```

`flatMap` chains steps that each might fail:

```scala
val users  = Map(1 -> "max")
val emails = Map("max" -> "max@example.com")

def emailFor(id: Int): Option[String] =
  users.get(id).flatMap(name => emails.get(name))

emailFor(1)    // Some(max@example.com)
emailFor(2)    // None, no user 2, so the second lookup never runs
```

The same with a `for` comprehension (topic 02), which reads more naturally when there
are several steps:

```scala
def emailFor(id: Int): Option[String] =
  for {
    name  <- users.get(id)
    email <- emails.get(name)
  } yield email
```

> **🐍 Python equivalent**
> ```python
> name = users.get(id)
> email = emails.get(name) if name is not None else None
> ```

## Option in your own functions

Return `Option` instead of `null` or a magic value like `-1`:

```scala
def safeDivide(a: Int, b: Int): Option[Int] =
  if (b == 0) None else Some(a / b)

safeDivide(10, 2)    // Some(5)
safeDivide(10, 0)    // None
```

## The flow

```
List / Map lookup
       ↓
returns Option
       ↓
  Some(v)          None
     ↓               ↓
getOrElse(d)    getOrElse(d)
     ↓               ↓
 returns v       returns d
```

## Quick reference

| Operation | Code | Result |
|-----------|------|--------|
| Create | `Some(55)` / `None` | |
| From nullable | `Option(x)` | `None` if `x` is null |
| Has value? | `opt.isDefined` / `opt.nonEmpty` | `true` / `false` |
| Empty? | `opt.isEmpty` | `true` / `false` |
| Get (risky) | `opt.get` | 55 or **crash** |
| Get safely | `opt.getOrElse(0)` | 55 or 0 |
| Find in list | `lst.find(_ > 2)` | `Some(3)` or `None` |
| Map lookup | `map.get(key)` | `Some(v)` or `None` |
| Transform | `opt.map(_ * 2)` | `Some(110)` or `None` |
| Filter | `opt.filter(_ > 10)` | `Some(55)` or `None` |
| Fallback | `opt.orElse(Some(99))` | `Some(55)` or `Some(99)` |
| Both cases | `opt.fold(default)(f)` | value |
| Pattern match | `case Some(v) => ... case None => ...` | |

> **🔀 2 vs 3**
> `Option` is the same in all versions. Scala 3 can also express "maybe null" with
> union types (`String | Null`) when the explicit-nulls compiler option is on, but `Option`
> is still the idiomatic choice.

> **🎯 Interview one-liner**
> "`Option` represents a value that may be absent: `Some(value)` or `None`. It replaces
> `null` and makes the missing case visible in the type, so the compiler forces you to
> handle it. Prefer `map`, `getOrElse` and pattern matching over `.get`."

## Practice

1. Write `def parseAge(s: String): Option[Int]` that returns `None` for non-numbers or negatives.
2. Given `Map[String, String]` of user → manager, write `managerOfManager(user)` with `flatMap`.
3. Sum a `List[Option[Int]]`, skipping the `None`s (hint: `flatten`).

---
Previous: [14. Collection operations](14-collection-operations.md) · Next: [18. DSA patterns](18-dsa-patterns.md) (16 and 17 coming soon)
