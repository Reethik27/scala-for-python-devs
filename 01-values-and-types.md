# 01. Values, Variables and Types

## `val` and `var`

```scala
val name = "Reetik"   // immutable: cannot be reassigned
var count = 0         // mutable: can be reassigned
count = count + 1     // fine
// name = "Bob"       // compile error: reassignment to val
```

> **🐍 Python equivalent**
> Python has no `val`. Every name can be reassigned:
> ```python
> name = "Reetik"
> count = 0
> count += 1
> ```
> `typing.Final` is the closest thing to `val`, but Python only warns about it in type checkers.

**Rule of thumb:** use `val` by default. Reach for `var` only when you really need it.
Idiomatic Scala code has very few `var`s.

## Type inference and annotations

Scala is **statically typed**, but you rarely have to write types because the compiler infers them.

```scala
val age = 30                // inferred: Int
val price: Double = 9.99    // explicit annotation
val ok: Boolean = true
```

The type goes **after** the name, like Python type hints:

```python
age: int = 30
```

The difference: in Python, hints are optional and not enforced at runtime.
In Scala, the compiler enforces them.

```scala
val n: Int = "hello"   // compile error: Found String, Required Int
```

## Basic types

| Scala | Python | Notes |
|-------|--------|-------|
| `Int` | `int` | 32-bit. Overflows silently! |
| `Long` | `int` | 64-bit. Write literals as `10L` |
| `BigInt` | `int` | Arbitrary size, like Python's `int` |
| `Double` | `float` | 64-bit floating point |
| `Float` | (none) | 32-bit, write as `1.5f` |
| `Boolean` | `bool` | `true` / `false` (lowercase) |
| `Char` | (none) | Single character in single quotes: `'a'` |
| `String` | `str` | Always double quotes: `"hi"` |
| `Unit` | `None` (as a return type) | "No meaningful value". Its only value is `()` |
| `Null` / `null` | `None` | Avoid; use `Option` instead (topic 11) |

> **⚠️ Gotcha: integer overflow**
> ```scala
> val big = Int.MaxValue + 1   // -2147483648, no error!
> ```
> Python's `int` never overflows. Use `Long` or `BigInt` for large numbers.

> **⚠️ Gotcha: integer division**
> ```scala
> 7 / 2      // 3   (like Python's 7 // 2)
> 7 / 2.0    // 3.5 (like Python's 7 / 2)
> ```

> **⚠️ Gotcha: quotes matter**
> `'a'` is a `Char`, `"a"` is a `String`. `'hello'` does not compile.

## Everything is an object

There are no "primitive" types in your code. Even `1` has methods:

```scala
1.to(5)          // Range 1 to 5
1 + 2            // actually 1.+(2)
"abc".length     // 3
(-5).abs         // 5
```

## Type hierarchy (simplified)

```
                  Any
          ┌────────┴────────┐
        AnyVal            AnyRef  (= java.lang.Object)
   Int, Double,         String, List, your classes...
   Boolean, Unit...              │
                                Null
          └────────┬────────┘
                Nothing
```

- `Any` is the top of everything, like Python's `object`.
- `Nothing` is the bottom: a type with no values. A function that always throws returns `Nothing`.

## `lazy val`

```scala
lazy val config = loadConfig()  // computed the first time it is used, then cached
```

> **🐍 Python equivalent:** `functools.cached_property`

## 🔀 2 vs 3

Values and basic types are the same in both versions. One Scala 3 addition worth knowing:

```scala
// Scala 3 only: union types
val id: Int | String = "abc"
```

> **🐍 Python equivalent:** `id: int | str = "abc"`

Scala 2 has no union types; you would use `Either[Int, String]` instead (topic 12).

## Practice

1. In the REPL, check the inferred type of `1`, `1L`, `1.0`, `'1'`, `"1"`.
2. Declare a `val` and try to reassign it. Read the compiler error.
3. Compute `Int.MaxValue + 1`, then the same with `BigInt(Int.MaxValue) + 1`.

---
Previous: [00. Setup](00-setup.md) · Next: [02. Control flow](02-control-flow.md)
