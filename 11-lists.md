# 11. Lists and ListBuffer

## List

A Scala `List` is an **immutable singly linked list**. You can't change it; every
"modification" returns a new list. That sounds slow, but the new list shares most of
its nodes with the old one, so prepending is O(1).

> **⚠️ Gotcha: Scala `List` ≠ Python `list`**
> Python's `list` is a resizable array. Scala's `List` is a linked list: fast at the
> front, slow for `xs(i)` and appending. The Scala equivalent of Python's `list` is
> `ArrayBuffer` (mutable) or `Vector` (immutable).

### Creating lists

```scala
val mylist: List[Int] = List(1, 2, 5, 8, 9, 6, 4)
val names = List("Max", "Tom", "John")      // List[String] inferred
val empty = List.empty[Int]                 // or List[Int]() or Nil

1 :: 5 :: 9 :: Nil        // List(1, 5, 9), built with "cons"
List.fill(5)(2)           // List(2, 2, 2, 2, 2)
List.range(1, 6)          // List(1, 2, 3, 4, 5)
(1 to 5).toList           // List(1, 2, 3, 4, 5)
List.tabulate(4)(i => i * 10) // List(0, 10, 20, 30)
```

`::` (pronounced "cons") puts an element **in front of** a list. `Nil` is the empty list.
`1 :: 5 :: 9 :: Nil` is read right to left: `1 :: (5 :: (9 :: Nil))`.

### Basic operations

```scala
val mylist = List(1, 2, 5, 8, 9, 6, 4)

mylist.head          // 1 (first element; throws on empty list)
mylist.tail          // List(2, 5, 8, 9, 6, 4) (everything but the first)
mylist.last          // 4
mylist.init          // List(1, 2, 5, 8, 9, 6)
mylist.isEmpty       // false
mylist.length        // 7
mylist(2)            // 5  (O(n), walks the list)
mylist.reverse       // List(4, 6, 9, 8, 5, 2, 1)
mylist.max           // 9
mylist.sum           // 35
mylist.contains(8)   // true
mylist.headOption    // Some(1), safe on empty lists (topic 15)
```

### Adding elements (always returns a new list)

```scala
val xs = List(1, 2, 3)

0 :: xs              // List(0, 1, 2, 3)   prepend, O(1)  ← preferred
0 +: xs              // same
xs :+ 4              // List(1, 2, 3, 4)   append, O(n)
xs ++ List(4, 5)     // List(1, 2, 3, 4, 5)
List(-1, 0) ::: xs   // List(-1, 0, 1, 2, 3)  list-specific concat
xs.updated(0, 99)    // List(99, 2, 3)
xs.filterNot(_ == 2) // List(1, 3), "remove" by value
xs                   // still List(1, 2, 3)
```

> **💡 Tip:** if you need to append in a loop, either prepend and `reverse` at the end,
> or use a `ListBuffer` (below).

### Looping

```scala
val mylist = List(10, 20, 30, 40, 50)

mylist.foreach(println)               // each element
for (x <- mylist) println(x)          // same

// Summing with a var (works, but not idiomatic)
var sum = 0
mylist.foreach(sum += _)              // sum == 150

// Idiomatic
mylist.sum                            // 150
mylist.foldLeft(0)(_ + _)             // 150

for ((x, i) <- mylist.zipWithIndex) println(s"$i: $x")
```

### Pattern matching on lists

Lists are built for recursion and `match`:

```scala
def sumList(xs: List[Int]): Int = xs match {
  case Nil          => 0
  case head :: tail => head + sumList(tail)
}

def describe(xs: List[Int]): String = xs match {
  case Nil           => "empty"
  case x :: Nil      => s"one element: $x"
  case x :: y :: _   => s"starts with $x, $y"
}
```

> **🐍 Python equivalent (3.10+)**
> ```python
> match xs:
>     case []: ...
>     case [x]: ...
>     case [x, y, *_]: ...
> ```

## Fixing the broken snippet from the notes

The original notes had this, which doesn't compile:

```scala
println(:: mylist);      // ✗ `::` needs an element on the left
var sum Int = 0;         // ✗ missing colon
def main(args: Array[String]) { ... }   // ✗ procedure syntax, removed in Scala 3
```

Corrected and runnable on Scala 2.12, 2.13 and 3:

```scala
object Demo {
  val mylist: List[Int] = List(1, 2, 5, 8, 9, 6, 4)
  val names: List[String] = List("Max", "Tom", "John")

  def main(args: Array[String]): Unit = {
    println(0 :: mylist)           // List(0, 1, 2, 5, 8, 9, 6, 4)
    println(mylist)                // List(1, 2, 5, 8, 9, 6, 4)
    println(names)                 // List(Max, Tom, John)
    println(1 :: 5 :: 9 :: Nil)    // List(1, 5, 9)
    println(mylist.head)           // 1
    println(names.tail)            // List(Tom, John)
    println(mylist.tail)           // List(2, 5, 8, 9, 6, 4)
    println(names.isEmpty)         // false
    println(mylist.reverse)        // List(4, 6, 9, 8, 5, 2, 1)
    println(List.fill(5)(2))       // List(2, 2, 2, 2, 2)
    println(mylist.max)            // 9
    mylist.foreach(println)

    var sum: Int = 0
    mylist.foreach(sum += _)
    println(sum)                   // 35
  }
}
```

Semicolons are optional in Scala; leave them out.

## ListBuffer

A `ListBuffer` is a mutable builder for lists: O(1) append **and** prepend, and
`.toList` at the end is O(1).

```scala
import scala.collection.mutable.ListBuffer

val lb = ListBuffer(1, 2, 3)
lb += 4                       // append  → ListBuffer(1, 2, 3, 4)
lb.prepend(0)                 // prepend → ListBuffer(0, 1, 2, 3, 4)
-1 +=: lb                     // also prepend (operator form) → ListBuffer(-1, 0, 1, 2, 3, 4)
lb.insert(2, 99)              // → ListBuffer(-1, 0, 99, 1, 2, 3, 4)
lb.remove(2)                  // returns 99
lb -= 3                       // remove by value → ListBuffer(-1, 0, 1, 2, 4)
lb.toList                     // List(-1, 0, 1, 2, 4)
```

Typical use: building a result, then returning an immutable `List`.

```scala
def evens(n: Int): List[Int] = {
  val out = ListBuffer[Int]()
  for (i <- 0 to n if i % 2 == 0) out += i
  out.toList
}
```

(The idiomatic one-liner is `(0 to n).filter(_ % 2 == 0).toList`.)

## Array vs List

| | `Array` | `List` |
|---|---|---|
| Mutability | Mutable: `arr(0) = 5` works | Immutable: no updates in place |
| Size | Fixed | Immutable; ops return a new list |
| Structure | Contiguous memory (Java array) | Linked nodes |
| Fast | Random access by index | Head, tail, prepend |
| Slow | Growing | Index access, append |
| `==` | Compares references ⚠️ | Compares contents |
| `println` | Prints `[I@1b6d...` ⚠️ | Prints `List(1, 2, 3)` |
| Use for | Updating values, DSA with indexes | Data that shouldn't change, recursion |

`foreach`, `for`, `map`, `filter` and the rest work on both, and on every other collection.

> **🔀 2 vs 3**
> `List` itself is the same in all versions. `0 +=: lb` still works in 2.13 and 3, but
> `lb.prepend(0)` reads more clearly. In 2.12, `ListBuffer.prepend` takes varargs
> (`lb.prepend(0, -1)`); in 2.13+ it takes one element and `prependAll` takes many.

> **🎯 Interview one-liner**
> "Scala's `List` is an immutable linked list: O(1) prepend and head, O(n) index and
> append. Use `::` to build it and pattern match with `head :: tail`. Use `ListBuffer`
> to build one efficiently, `Vector` for fast immutable indexing, and `ArrayBuffer` for
> a mutable Python-style list."

## Practice

1. Write a recursive `def maxOf(xs: List[Int]): Int` using pattern matching.
2. Reverse a list without `.reverse`, using `foldLeft` and `::`.
3. Use a `ListBuffer` to collect all words longer than 3 letters from a sentence.

---
Previous: [10. Arrays and ArrayBuffer](10-arrays.md) · Next: [12. Sets](12-sets.md)
