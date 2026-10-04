# 13. Maps

A `Map` is a collection of **key → value** pairs, like Python's `dict`:

- every key is unique,
- each key points to exactly one value,
- it is **immutable by default** (operations return a new map),
- keys and values can be any type.

## Creating maps

```scala
val mymap: Map[Int, String] = Map(801 -> "Max", 802 -> "Tom", 804 -> "July")
val mymap2 = Map(805 -> "Lua")
val empty = Map.empty[String, Int]

List(("a", 1), ("b", 2)).toMap        // Map(a -> 1, b -> 2)
List("x", "y").zip(List(1, 2)).toMap  // Map(x -> 1, y -> 2), like dict(zip(...))
```

`801 -> "Max"` is just a nicer way to write the tuple `(801, "Max")`.

> **🐍 Python equivalent**
> ```python
> mymap = {801: "Max", 802: "Tom", 804: "July"}
> ```

## Reading values

```scala
mymap(801)                   // "Max"
// mymap(999)                // NoSuchElementException: key not found ← crash
mymap.get(801)               // Some("Max")
mymap.get(999)               // None (safe, see topic 15)
mymap.getOrElse(999, "N/A")  // "N/A"
mymap.contains(801)          // true

mymap.keys                   // Set(801, 802, 804) (Iterable in 2.13+)
mymap.keySet                 // Set(801, 802, 804)
mymap.values                 // Iterable(Max, Tom, July)
mymap.size                   // 3
mymap.isEmpty                // false
Map().isEmpty                // true
```

> **🐍 Python equivalent**
> | Python | Scala |
> |--------|-------|
> | `d[k]` | `m(k)` |
> | `d.get(k)` | `m.get(k)` (returns `Option`) |
> | `d.get(k, default)` | `m.getOrElse(k, default)` |
> | `k in d` | `m.contains(k)` |
> | `len(d)` | `m.size` |

## Adding, updating and removing (returns a new map)

```scala
val newMap = mymap + (806 -> "Sara")     // add a pair
// Map(801 -> Max, 802 -> Tom, 804 -> July, 806 -> Sara)

mymap + (801 -> "Maximilian")            // same key: value is replaced
mymap.updated(801, "Maximilian")         // same thing

mymap ++ mymap2                          // merge; right side wins on duplicate keys

mymap - 802                              // Map(801 -> Max, 804 -> July)
mymap -- List(801, 802)                  // Map(804 -> July), remove several
```

> **🔀 2 vs 3**
> The notes used `mymap - (801, 802)` to remove two keys. That works on 2.12 but is
> deprecated in 2.13 and Scala 3. Use `mymap -- List(801, 802)`, which works everywhere.

## Looping

```scala
// Over key-value pairs (preferred)
for ((key, value) <- mymap) println(s"Key: $key Value: $value")

mymap.foreach { case (key, value) =>
  println(s"Key: $key Value: $value")
}

// Over keys only
mymap.keys.foreach { key =>
  println(s"Key: $key")
  println(s"Value: ${mymap(key)}")
}
```

> **🐍 Python equivalent:** `for key, value in d.items():`

> **🔀 2 vs 3**
> Scala 3 lets you drop the `case` in a lambda that takes a tuple:
> `mymap.foreach((key, value) => println(s"$key $value"))`. Scala 2 needs `{ case (k, v) => ... }`.

## Transforming maps

```scala
val ages = Map("Max" -> 28, "Tom" -> 35, "Ana" -> 22)

ages.filter { case (_, age) => age > 25 }  // Map(Max -> 28, Tom -> 35)
ages.filter(_._2 > 25)                     // same, using tuple accessors
ages.map { case (name, age) => name -> (age + 1) }   // add 1 to every age
ages.map { case (name, age) => age -> name }         // swap keys and values
ages.maxBy(_._2)                           // (Tom, 35)
ages.toList.sortBy(_._2)                   // List((Ana,22), (Max,28), (Tom,35))
ages.values.sum                            // 85
```

`_._1` is the key, `_._2` is the value (tuples are 1-indexed).

### `mapValues`: transform only the values

```scala
val names = Map(1 -> "Tom", 2 -> "Max", 3 -> "John")

// Scala 2.12
names.mapValues(x => "hi " + x)

// Scala 2.13 and 3
names.view.mapValues(x => "hi " + x).toMap
names.map { case (k, v) => k -> ("hi " + v) }   // works in every version
// Map(1 -> hi Tom, 2 -> hi Max, 3 -> hi John)
```

> **🔀 2 vs 3**
> In 2.12, `mapValues` returns a *lazy wrapper* that re-runs your function every time
> you read a value (a famous source of bugs). In 2.13 it's deprecated and moved to
> `.view.mapValues(...)`; add `.toMap` to get a real map. `filterKeys` changed the same way.

## Counting and grouping (very common in interviews)

```scala
val words = List("apple", "bob", "apple", "cat", "bob", "apple")

// Count occurrences: like collections.Counter
words.groupBy(identity).map { case (w, ws) => w -> ws.size }
// Map(apple -> 3, bob -> 2, cat -> 1)

words.groupMapReduce(identity)(_ => 1)(_ + _)  // same result, 2.13+ and Scala 3 only

// Group by a property
words.distinct.groupBy(_.length)
// Map(5 -> List(apple), 3 -> List(bob, cat))
```

## Default values

```scala
val counts = Map("a" -> 1).withDefaultValue(0)
counts("a")     // 1
counts("zzz")   // 0 instead of a crash, like defaultdict(int)
```

## Mutable maps

```scala
import scala.collection.mutable

val m = mutable.Map(1 -> "a", 2 -> "b")
m(1) = "X"                     // update existing key
m(3) = "c"                     // add new key
m += (4 -> "d")                // add a pair
m -= 2                         // remove key 2
m.remove(4)                    // remove and return Some("d")
println(m)                     // HashMap(1 -> X, 3 -> c)

m.getOrElseUpdate(5, "e")      // get, or insert and return "e" if missing
m.update(1, "Y")               // same as m(1) = "Y"
m.clear()
```

Mutable maps are what you'll use for most DSA problems:

```scala
// Character counts (Python: Counter(s))
val counts = mutable.Map[Char, Int]()
for (ch <- "banana") counts(ch) = counts.getOrElse(ch, 0) + 1
// counts: a -> 3, n -> 2, b -> 1

// With a default, like defaultdict(int)
val counts2 = mutable.Map[Char, Int]().withDefaultValue(0)
for (ch <- "banana") counts2(ch) += 1
```

> **⚠️ Gotcha**
> `withDefaultValue` only affects *reads*; it doesn't store the default.
> `counts2(ch) += 1` works because it expands to `counts2(ch) = counts2(ch) + 1`.

> **🐍 Python equivalent**
> | Python | Scala `mutable.Map` |
> |--------|---------------------|
> | `d[k] = v` | `m(k) = v` |
> | `del d[k]` | `m -= k` or `m.remove(k)` |
> | `d.setdefault(k, v)` | `m.getOrElseUpdate(k, v)` |
> | `defaultdict(int)` | `mutable.Map[K, Int]().withDefaultValue(0)` |
> | `Counter(xs)` | `xs.groupBy(identity).map { case (k, v) => k -> v.size }` |
> | `d.items()` | `m` itself (iterating gives `(k, v)` tuples) |

## Sorted maps

```scala
import scala.collection.immutable.TreeMap
val tm = TreeMap(3 -> "c", 1 -> "a", 2 -> "b")
tm          // TreeMap(1 -> a, 2 -> b, 3 -> c), always sorted by key
tm.head     // (1, a)
```

A plain `Map` has no guaranteed order, just like `Set`.

## Quick reference

| Operation | Code | Result |
|-----------|------|--------|
| Access by key | `m(801)` | `Max` (crashes if missing) |
| Safe access | `m.get(801)` | `Some(Max)` / `None` |
| With default | `m.getOrElse(99, "N/A")` | `N/A` |
| All keys | `m.keys` | `Set(801, 802, 804)` |
| All values | `m.values` | `Iterable(Max, Tom, July)` |
| Key exists | `m.contains(801)` | `true` |
| Size / empty | `m.size` / `m.isEmpty` | `3` / `false` |
| Add pair | `m + (806 -> "Sara")` | new map |
| Remove key | `m - 802` | new map |
| Merge | `m ++ m2` | combined map |
| Filter | `m.filter(_._1 > 801)` | filtered map |
| Transform values | `m.map { case (k, v) => k -> f(v) }` | new map |
| Loop | `for ((k, v) <- m)` | each pair |

> **🎯 Interview one-liner**
> "A Map stores unique keys, each pointing to one value. It's immutable by default, so
> `+` and `-` return new maps. Use `get` or `getOrElse` for safe lookups, and
> `mutable.Map` with `getOrElseUpdate` or `withDefaultValue` for counting problems."

## Practice

1. Count word frequencies in a sentence and print the top 3.
2. Invert a `Map[String, Int]` into a `Map[Int, List[String]]` with `groupBy`.
3. Valid anagram (LeetCode 242) with a `mutable.Map[Char, Int]`. Solution in [topic 18](18-dsa-patterns.md).

---
Previous: [12. Sets](12-sets.md) · Next: [14. Collection operations](14-collection-operations.md)
