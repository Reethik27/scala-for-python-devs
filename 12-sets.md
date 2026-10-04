# 12. Sets

A `Set` holds **unique** elements with **no guaranteed order**. It is immutable by
default: `+`, `++`, `-` and friends return a new set. Use a set for fast membership
checks (`contains` is O(1)) or to remove duplicates.

> **🐍 Python equivalent:** `set` (mutable) / `frozenset` (immutable).
> Scala's default `Set` behaves like `frozenset`; `mutable.Set` behaves like `set`.

## Creating sets

```scala
val myset: Set[Int] = Set(1, 2, 5, 8, 9, 6, 4)   // explicit type
val myset2 = Set(4, 2, 9, 18, 19, 16, 14)          // inferred
val names: Set[String] = Set("Max", "Tom", "John")
val empty = Set.empty[Int]

List(1, 1, 2, 3, 3).toSet     // Set(1, 2, 3), removes duplicates
```

## Checking membership

```scala
names.contains("Max")    // true
names("Max")             // true, a Set can be called like a function
names("Maxxxx")          // false
```

> **🐍 Python equivalent:** `"Max" in names`

## Adding and removing (returns a new set)

```scala
val s = Set(1, 2, 3)
s + 4                    // Set(1, 2, 3, 4)
s + 2                    // Set(1, 2, 3), already there, no change
s - 2                    // Set(1, 3)
s ++ List(4, 5)          // Set(5, 1, 2, 3, 4), add many
s -- List(1, 2)          // Set(3), remove many
s                        // still Set(1, 2, 3)
```

## Set algebra

```scala
val a = Set(1, 2, 5, 8, 9, 6, 4)
val b = Set(4, 2, 9, 18, 19, 16, 14)

// Union: everything in either
a ++ b                   // Set(1, 2, 4, 5, 6, 8, 9, 14, 16, 18, 19) (order varies)
a | b                    // same
a.union(b)               // same

// Intersection: only in both
a & b                    // Set(2, 4, 9)
a.intersect(b)           // same

// Difference: in a but not in b
a &~ b                   // Set(1, 5, 6, 8)
a.diff(b)                // same
a -- b                   // same

// Subset check
Set(2, 4).subsetOf(a)    // true
```

> **🐍 Python equivalent**
> | Python | Scala |
> |--------|-------|
> | `a \| b` | `a \| b` or `a ++ b` |
> | `a & b` | `a & b` |
> | `a - b` | `a &~ b`, `a diff b` or `a -- b` |
> | `a ^ b` | `(a diff b) ++ (b diff a)` |
> | `a <= b` | `a.subsetOf(b)` |

## Other useful methods

```scala
val s = Set(1, 2, 3, 4, 5)

s.size                  // 5
s.isEmpty               // false
s.max / s.min           // 5 / 1
s.sum                   // 15
s.filter(_ > 2)         // Set(3, 4, 5)
s.map(_ * 2)            // Set(2, 4, 6, 8, 10)
s.map(_ % 2)            // Set(1, 0), duplicates collapse!
s.exists(_ > 4)         // true
s.toList                // List(5, 1, 2, 3, 4) (some order)
s.toList.sorted         // List(1, 2, 3, 4, 5)
s.mkString(", ")        // "5, 1, 2, 3, 4" (some order)
```

## Looping

```scala
names.foreach(println)
for (name <- names) println(name)
```

> **⚠️ Gotcha: order is not guaranteed**
> `Set(1, 2, 5, 8, 9, 6, 4).head` is *some* element, not necessarily `1`.
> Small sets (up to 4 elements) happen to keep insertion order, which makes this
> confusing: code that works on 3 elements can break on 10.
> Don't rely on `head`, `tail` or print order. Sort first, or use a `SortedSet`.
>
> On 2.13 and Scala 3, sets with more than 4 elements print as `HashSet(...)`
> instead of `Set(...)`. It's the same type of set; only the name in the output differs.

## Sorted sets

```scala
import scala.collection.immutable.SortedSet
val ss = SortedSet(5, 1, 3)    // SortedSet(1, 3, 5)
ss.head                        // 1, always the smallest
ss.last                        // 5
ss.range(2, 5)                 // SortedSet(3), elements in [2, 5)
```

## Mutable sets

```scala
import scala.collection.mutable

val ms = mutable.Set(1, 2, 3)
ms += 4                 // add          → Set(1, 2, 3, 4)
ms -= 1                 // remove       → Set(2, 3, 4)
ms ++= List(7, 8)       // add many
ms.add(4)               // false: 4 was already there
ms.add(10)              // true: newly added
ms.remove(2)            // true: it was present
ms.remove(99)           // false
ms.clear()

val ts = mutable.TreeSet(5, 1, 3)   // mutable sorted set
ts.head                             // 1
ts.last                             // 5
```

> **🐍 Python equivalent:** `add` / `discard` / `update`. Note that Scala's `add` and
> `remove` **return a Boolean** telling you whether anything changed, which is handy
> in "have I seen this before?" loops:
> ```scala
> val seen = mutable.Set[Int]()
> for (x <- List(3, 1, 3)) if (!seen.add(x)) println(s"duplicate: $x")
> ```

## Quick reference

| Operation | Code | Result |
|-----------|------|--------|
| Add element | `s + 10` | New set with 10 |
| Remove element | `s - 5` | New set without 5 |
| Exists? | `s.contains(3)` or `s(3)` | `true` / `false` |
| Union | `a ++ b` / `a \| b` | All elements |
| Intersection | `a & b` / `a.intersect(b)` | Common elements |
| Difference | `a -- b` / `a.diff(b)` / `a &~ b` | In `a`, not in `b` |
| Size | `s.size` | Int |
| Is empty | `s.isEmpty` | `true` / `false` |
| Max / Min | `s.max` / `s.min` | Largest / smallest |
| Filter | `s.filter(_ > 3)` | Matching elements |
| Sorted | `s.toList.sorted` or `SortedSet` | In order |

> **🔀 2 vs 3**
> Sets work the same in 2.12, 2.13 and 3. Two small 2.13 changes: `s + (1, 2)` (adding
> several at once with `+`) is deprecated, so use `s ++ Set(1, 2)`; and
> `mutable.Set.retain` was renamed `filterInPlace`.

> **🎯 Interview one-liner**
> "A Set stores unique elements with no guaranteed order. It's immutable by default,
> so `+`, `++` and `diff` return new sets. Use it for O(1) membership checks or to
> remove duplicates; use `SortedSet`/`TreeSet` when you need order."

## Practice

1. Find the common elements of two lists using sets.
2. Return `true` if an array contains any duplicate (LeetCode 217).
3. Find the words that appear in sentence A but not sentence B.

---
Previous: [11. Lists](11-lists.md) · Next: [13. Maps](13-maps.md)
