# 05. Classes and Objects

A **class** is a blueprint: it defines the data (fields) and behaviour (methods) its
instances have. Scala classes are much shorter than Java or Python ones because the
constructor is part of the class header.

## A first class

```scala
// Scala 2 and 3
class User(private var name: String, val age: Int) {
  def printName(): Unit = println(name)
  def greet(other: String): String = s"Hi $other, I'm $name"
}

val user = new User("Max", 28)   // Scala 3 also allows: User("Max", 28)
user.printName()                 // Max
println(user.age)                // 28
```

```scala
// Scala 3 indentation style
class User(private var name: String, val age: Int):
  def printName(): Unit = println(name)
  def greet(other: String): String = s"Hi $other, I'm $name"
```

> **🐍 Python equivalent**
> ```python
> class User:
>     def __init__(self, name: str, age: int):
>         self._name = name
>         self.age = age
>     def print_name(self):
>         print(self._name)
> ```
> No `self` parameter and no `__init__`: the parameters in the class header *are* the constructor.

> **🔀 2 vs 3**
> - `def printName { println(name) }` (procedure syntax) works in Scala 2.12, is
>   deprecated in 2.13 and **does not compile in Scala 3**. Write `def printName(): Unit = ...`.
> - Scala 3 lets you drop `new`: `User("Max", 28)`. Scala 2 needs `new` (except for case classes).

## Constructor parameters: plain, `val`, `var`, `private`

The keyword in front of a constructor parameter decides whether outside code can read
or change it:

```scala
class A(name: String)          // only usable inside the class, no getter
class B(val name: String)      // public getter, read-only
class C(var name: String)      // public getter and setter
class D(private var name: String) // mutable, but only inside the class
```

| Constructor param | Read from outside? | Change it? |
|-------------------|--------------------|------------|
| `name: String` | No | No |
| `val name: String` | Yes | No |
| `var name: String` | Yes | Yes, from anywhere |
| `private val name: String` | No | No |
| `private var name: String` | No | Only inside the class |

```scala
val b = new B("Max")
b.name            // "Max"
// b.name = "Tom" // error: reassignment to val

val c = new C("Max")
c.name = "Tom"    // fine
c.name            // "Tom"

val d = new D("Max")
// d.name         // error: name cannot be accessed
```

> **🐍 Python comparison**
> Python's `_name` is only a convention. Scala's `private` is **enforced by the compiler**.

## `val user` vs `var user`

```scala
val user = new User("Max", 28)   // the reference can't point elsewhere
var user2 = new User("Max", 28)  // can be reassigned to another User
```

`val` only freezes the *reference*. If the class has `var` fields, the object itself
can still change. Prefer `val` for references.

## Fields and methods in the body

```scala
class Counter(start: Int) {
  private var count = start        // field
  def increment(): Unit = count += 1
  def current: Int = count         // no parentheses: reads like a field
}

val c = new Counter(10)
c.increment()
c.current                          // 11
```

**Convention:** methods with side effects get `()` (`increment()`); pure getters
don't (`current`).

## Auxiliary constructors and defaults

Default parameter values usually replace extra constructors:

```scala
class Point(val x: Int = 0, val y: Int = 0)

new Point()          // (0, 0)
new Point(5)         // (5, 0)
new Point(y = 3)     // (0, 3)
```

## `toString`, `override`

```scala
class Point(val x: Int, val y: Int) {
  override def toString: String = s"Point($x, $y)"
}
println(new Point(1, 2))   // Point(1, 2)
```

> **🐍 Python equivalent:** `__repr__` / `__str__`. Scala requires the `override` keyword.

## Inheritance

```scala
class Animal(val name: String) {
  def sound: String = "..."
}

class Dog(name: String) extends Animal(name) {
  override def sound: String = "Woof"
}

new Dog("Rex").sound   // "Woof"
```

A class can extend only **one** class. For multiple inheritance, Scala uses traits (topic 07).

## `object`: singletons

An `object` is a class with exactly one instance, created on first use.

```scala
object MathUtils {
  val Pi = 3.14159
  def square(x: Int): Int = x * x
}

MathUtils.square(4)   // 16
```

> **🐍 Python equivalent:** a module with functions and constants. Scala has no
> "static" keyword; you put those things in an `object`.

## Companion objects

An `object` with the **same name** as a class, in the same file, is its companion.
They can see each other's private members. It is the usual home for factory methods:

```scala
class Temperature private (val celsius: Double)   // private constructor

object Temperature {
  def fromCelsius(c: Double): Temperature    = new Temperature(c)
  def fromFahrenheit(f: Double): Temperature = new Temperature((f - 32) * 5 / 9)
}

val t = Temperature.fromFahrenheit(212)
t.celsius   // 100.0
```

> **🐍 Python equivalent:** `@classmethod` alternative constructors.

## Case classes preview

For plain data holders, use a **case class** (topic 06). It generates `toString`,
equality, `copy` and pattern matching support for you:

```scala
case class Person(name: String, age: Int)
val p = Person("Max", 28)     // no `new` needed, even in Scala 2
p                             // Person(Max,28)
p == Person("Max", 28)        // true
```

> **🐍 Python equivalent:** `@dataclass(frozen=True)`

## Quick reference

```
class User(private var name: String, val age: Int)
           ↑        ↑                ↑
       hidden    mutable         read-only
       outside   inside class    public getter
```

| Want | Write |
|------|-------|
| Method returning nothing | `def log(): Unit = ...` |
| Method returning a value | `def area: Double = ...` |
| Create an instance | `new User("Max", 28)` (Scala 3: `User("Max", 28)`) |
| Fixed reference | `val user = ...` |
| Static-like helpers | `object` |
| Factory methods | companion `object` |
| Data holder | `case class` |

> **🎯 Interview one-liner**
> "In Scala the primary constructor lives in the class header. `val` parameters become
> read-only fields, `var` parameters get a setter, and plain parameters stay private to
> the class. Singletons and static members go in an `object`."

## Practice

1. Write a `BankAccount` class with a private balance, `deposit` and `withdraw` methods.
2. Give it a companion object with `def empty(): BankAccount`.
3. Rewrite `Point` as a case class and compare `==` with the plain class version.

---
Previous: [04. Strings](04-strings.md) · Next: [09. Collections overview](09-collections-overview.md) (06 to 08 coming soon)
