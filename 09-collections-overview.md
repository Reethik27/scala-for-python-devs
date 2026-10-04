# 09. Collections Overview: Which One Do I Use?

Python has one go-to sequence (`list`), one set and one dict. Scala has more choices,
split into **immutable** (the default) and **mutable** versions. This page helps you pick;
the next pages cover each one.

## The short answer

| You want... | Python | Scala (immutable, default) | Scala (mutable) |
|-------------|--------|----------------------------|-----------------|
| A sequence | `list` | `List` or `Vector` | `ArrayBuffer` |
| Fixed-size, fast index | `list` / `array` | `Vector` | `Array` |
| Unique items | `set` | `Set` | `mutable.Set` |
| Key → value | `dict` | `Map` | `mutable.Map` |
| Sorted unique items | `sorted(set)` | `TreeSet` / `SortedSet` | `mutable.TreeSet` |
| A queue / deque | `collections.deque` | `Queue` | `mutable.Queue`, `ArrayDeque` (2.13+) |
| A stack | `list.append/pop` | `List` (prepend/head) | `mutable.Stack` or `ArrayBuffer` |
| A heap | `heapq` | (none) | `mutable.PriorityQueue` |
| Optional value | `None` or value | `Option` (topic 15) | |
| Fixed group of values | `tuple` | `(1, "a")` tuple | |

## Immutable vs mutable

```scala
val xs = List(1, 2, 3)
val ys = xs :+ 4          // NEW list List(1, 2, 3, 4); xs is unchanged

import scala.collection.mutable
val buf = mutable.ArrayBuffer(1, 2, 3)
buf += 4                  // changes buf in place
```

The immutable ones are imported automatically. For the mutable ones, write
`import scala.collection.mutable` and then use `mutable.Map`, `mutable.Set` and so on.
That prefix makes it obvious when something can change.

> **⚠️ Gotcha: `val` does not mean immutable contents**
> `val buf = mutable.ArrayBuffer(1)` can still grow with `buf += 2`.
> `val` only stops `buf` from pointing at a different buffer.

## Performance cheat sheet

| Collection | Mutable? | Best for | Fast | Slow |
|------------|----------|----------|------|------|
| `Array` | elements yes, size no | fixed-size work, two-pointer problems | index read/write O(1) | append (copies) |
| `ArrayBuffer` | yes | **Python `list` replacement, default for DSA** | index O(1), append O(1) | insert at front O(n) |
| `ListBuffer` | yes | building a `List` step by step | append and prepend O(1), `.toList` O(1) | index O(n) |
| `List` | no | recursion, pattern matching | prepend and `head` O(1) | append and index O(n) |
| `Vector` | no | general immutable sequence | index, append, prepend ~O(1) | |
| `Set` / `mutable.Set` | either | membership checks, removing duplicates | `contains` O(1) | |
| `Map` / `mutable.Map` | either | lookups by key | get/put O(1) | |

## Every collection shares the same methods

`map`, `filter`, `foldLeft`, `foreach`, `exists`, `groupBy`, `sortBy`, `mkString`...
work on all of them (and on `String`). Learn them once in
[topic 14](14-collection-operations.md) and use them everywhere.

## Converting between collections

```scala
val arr = Array(3, 1, 2)
arr.toList          // List(3, 1, 2)
arr.toVector
arr.toSet           // Set(3, 1, 2)
arr.toBuffer        // mutable buffer
List(("a", 1), ("b", 2)).toMap   // Map(a -> 1, b -> 2)
Map("a" -> 1).toList             // List((a,1))

arr.to(List)        // Scala 2.13 / 3 generic form
// arr.to[List]     // Scala 2.12 form
```

> **🔀 2 vs 3 (really 2.12 vs 2.13+)**
> Scala **2.13 rewrote the collections library**, and Scala 3 uses the 2.13 version.
> Most code looks the same, but a few things changed. Each collection page has a
> 🔀 box with the details. The big ones:
> - `.to[List]` became `.to(List)`
> - `mapValues` / `filterKeys` now return a lazy view (use `.view.mapValues(f).toMap`)
> - `Stream` became `LazyList`
> - `ArrayDeque` was added; `mutable.Stack` and `mutable.Queue` were reimplemented on top of it
> - `groupMapReduce`, `toIntOption`, `tapEach` and others were added

---
Previous: [05. Classes and objects](05-classes-and-objects.md) · Next: [10. Arrays and ArrayBuffer](10-arrays.md)
