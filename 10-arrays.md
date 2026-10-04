# 10. Arrays and ArrayBuffer

## Array

An `Array` is a **fixed-size, mutable** sequence of elements of one type. It is a
plain Java array underneath, so indexing is very fast.

- **Fixed size:** once created, its length never changes.
- **Mutable:** you can overwrite the values inside.
- **One type:** an `Array[Int]` holds only `Int`s.

### Creating arrays

```scala
// 1. Empty array of a given size, filled with default values
val a: Array[Int] = new Array[Int](4)      // Array(0, 0, 0, 0)
val flags = new Array[Boolean](5)          // Array(false, false, false, false, false)
val names = new Array[String](2)           // Array(null, null)

// 2. From values (most common)
val nums = Array(1, 2, 3, 4, 5, 6, 8)

// 3. With a helper
Array.fill(5)(0)               // Array(0, 0, 0, 0, 0)
Array.fill(3)("hello")         // Array(hello, hello, hello)
Array.tabulate(5)(i => i * i)  // Array(0, 1, 4, 9, 16)
Array.range(0, 10, 2)          // Array(0, 2, 4, 6, 8)
Array.ofDim[Int](3, 3)         // 3x3 grid of zeros
```

> **🐍 Python equivalent**
> | Python | Scala |
> |--------|-------|
> | `[0] * 5` | `Array.fill(5)(0)` |
> | `[i * i for i in range(5)]` | `Array.tabulate(5)(i => i * i)` |
> | `[[0] * 3 for _ in range(3)]` | `Array.ofDim[Int](3, 3)` |

### Reading and writing

```scala
val arr = Array(20, 50, 10, 30)
arr(0)            // 20   ← parentheses, not square brackets
arr(0) = 99       // update in place
arr.length        // 4
arr.last          // 30
// arr(10)        // ArrayIndexOutOfBoundsException
// arr(-1)        // also an exception: no negative indexing!
```

> **⚠️ Gotcha: no negative indexes**
> Python's `arr[-1]` is `arr.last` or `arr(arr.length - 1)` in Scala.

### 2D arrays

```scala
val grid = Array.ofDim[Int](3, 3)
grid(0)(0) = 1
grid(1)(1) = 5
grid(2)(2) = 9          // access with (row)(col)

grid.map(_.mkString(" ")).foreach(println)
// 1 0 0
// 0 5 0
// 0 0 9
```

### Looping

```scala
val arr = Array(20, 50, 10, 30)

for (x <- arr) println(x)                    // each value
arr.foreach(println)                         // same thing

for (i <- 0 until arr.length) println(arr(i)) // each index (until excludes the end)
for (i <- arr.indices) println(arr(i))        // cleanest index loop

for ((x, i) <- arr.zipWithIndex) println(s"$i: $x")  // like Python's enumerate
```

> `0 to (arr.length - 1)` works too, but `0 until arr.length` or `arr.indices` is
> cleaner and avoids off-by-one mistakes.

### Printing an array

> **⚠️ Gotcha**
> ```scala
> println(Array(1, 2, 3))            // [I@6d06d69c  ← memory address, not the values!
> println(Array(1, 2, 3).mkString(", "))  // 1, 2, 3
> println(Array(1, 2, 3).toList)     // List(1, 2, 3)
> ```
> Arrays are Java arrays, so they don't have a readable `toString`. (The REPL is smarter and shows the contents.)

### Comparing arrays

> **⚠️ Gotcha**
> ```scala
> Array(1, 2) == Array(1, 2)            // false! compares references
> Array(1, 2).sameElements(Array(1, 2)) // true
> List(1, 2) == List(1, 2)              // true, Lists compare contents
> ```

### Common operations

These return a **new** array; the original is untouched (except `arr(i) = ...`).

```scala
val arr = Array(5, 3, 8, 1, 9, 2)

arr.length                  // 6
arr.sum / arr.min / arr.max // 28 / 1 / 9
arr.sorted                  // Array(1, 2, 3, 5, 8, 9)
arr.sortWith(_ > _)         // Array(9, 8, 5, 3, 2, 1) descending
arr.sorted.reverse          // same, descending
arr.reverse                 // Array(2, 9, 1, 8, 3, 5)
arr.contains(8)             // true
arr.indexOf(8)              // 2 (-1 if missing)
arr.head / arr.last         // 5 / 2
arr.tail                    // Array(3, 8, 1, 9, 2)
arr.init                    // Array(5, 3, 8, 1, 9), all but last
arr.isEmpty                 // false
arr.slice(1, 4)             // Array(3, 8, 1), end index excluded (arr[1:4])
arr.take(2) / arr.drop(2)   // Array(5, 3) / Array(8, 1, 9, 2)
arr.mkString(", ")          // "5, 3, 8, 1, 9, 2"
arr.toList                  // List(5, 3, 8, 1, 9, 2)
arr.clone()                 // independent copy

// "Changing" size: these build a new array
arr :+ 7                    // append  → Array(5, 3, 8, 1, 9, 2, 7)
0 +: arr                    // prepend → Array(0, 5, 3, 8, 1, 9, 2)
arr ++ Array(10, 11)        // concatenate
arr.patch(2, Seq(99), 0)    // insert 99 at index 2 → Array(5, 3, 99, 8, 1, 9, 2)
arr.patch(2, Nil, 1)        // remove index 2      → Array(5, 3, 1, 9, 2)
arr.updated(0, 100)         // copy with arr(0) = 100
arr.filterNot(_ == 8)       // remove by value     → Array(5, 3, 1, 9, 2)
```

**Memory trick for `:+` and `+:`:** the colon is on the side of the collection.
`arr :+ 7` (collection first), `0 +: arr` (collection last).

### Merging arrays

```scala
val a = Array(20, 50, 10, 30)
val b = Array(1, 2, 3)

a ++ b                  // Array(20, 50, 10, 30, 1, 2, 3)   ← preferred
Array.concat(a, b)      // same

import Array._          // brings concat, fill, ofDim... into scope
concat(a, b)            // same
```

### Swapping (handy for two-pointer problems)

```scala
def swap(arr: Array[Int], i: Int, j: Int): Unit = {
  val tmp = arr(i)
  arr(i) = arr(j)
  arr(j) = tmp
}
```

Scala has no `a, b = b, a` for array slots.

## ArrayBuffer: the Python `list` you were looking for

`ArrayBuffer` is a **resizable** array: fast index access and fast append. If you are
translating Python code that uses `list.append` / `list.pop`, use this.

```scala
import scala.collection.mutable.ArrayBuffer

val buf = ArrayBuffer(5, 3, 8)
buf += 7                      // append           → (5, 3, 8, 7)
buf.append(10)                // same as +=     → (5, 3, 8, 7, 10)
buf.prepend(0)                // add at front     → (0, 5, 3, 8, 7, 10)
buf.insert(2, 99)             // insert at 2      → (0, 5, 99, 3, 8, 7, 10)
buf.remove(2)                 // remove at 2, RETURNS 99
buf.remove(buf.length - 1)    // pop last, returns 10
buf -= 8                      // remove first 8 (by value) → (0, 5, 3, 7)
buf(0) = 42                   // update           → (42, 5, 3, 7)
buf ++= Seq(1, 2)             // append many      → (42, 5, 3, 7, 1, 2)
buf.remove(1, 2)              // remove 2 from index 1 → (42, 7, 1, 2)
buf.last                      // peek last
buf.isEmpty
buf.clear()                   // empty it
buf.toArray / buf.toList      // convert
```

> **🐍 Python equivalent**
> | Python `list` | `ArrayBuffer` |
> |---------------|---------------|
> | `lst.append(x)` | `buf += x` or `buf.append(x)` |
> | `lst.extend(ys)` | `buf ++= ys` |
> | `lst.insert(i, x)` | `buf.insert(i, x)` |
> | `lst.pop()` | `buf.remove(buf.length - 1)` |
> | `lst.pop(i)` | `buf.remove(i)` |
> | `lst.remove(x)` | `buf -= x` |
> | `lst[-1]` | `buf.last` |
> | `lst.clear()` | `buf.clear()` |

> **🔀 2 vs 3**
> In Scala 2.13 / 3 you can write `buf.dropRightInPlace(1)` to pop without returning,
> and `buf.prependAll`/`appendAll`. On 2.12, use `buf.remove(buf.length - 1)` and `++=` / `++=:`.

## Array vs ArrayBuffer vs List

| | `Array` | `ArrayBuffer` | `List` |
|---|---|---|---|
| Size | Fixed | Grows | Immutable (ops return new list) |
| Change elements | Yes | Yes | No |
| Index access | O(1) | O(1) | O(n) |
| Append | O(n) copy | O(1) | O(n) |
| Prepend | O(n) copy | O(n) | O(1) |
| `==` compares | references ⚠️ | contents | contents |
| Typical use | DSA with fixed size, Java interop | building results, stacks | functional code, recursion |

> **🎯 Interview one-liner**
> "Arrays in Scala are fixed-size, mutable and zero-indexed Java arrays. Use `Array(...)`
> for initial values or `new Array[T](n)` for defaults. When you need to grow, use
> `ArrayBuffer`; when you want immutability, use `List` or `Vector`."

## Practice

1. Reverse an `Array[Int]` in place using two pointers and `swap`.
2. Build an `ArrayBuffer` of the squares of 1 to 10, then pop the last three.
3. Create a 5x5 multiplication table with `Array.tabulate(5, 5)((i, j) => (i + 1) * (j + 1))`.

---
Previous: [09. Collections overview](09-collections-overview.md) · Next: [11. Lists and ListBuffer](11-lists.md)
