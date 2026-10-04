# 14. Collection Operations (the full toolbox)

Every Scala collection (`List`, `Vector`, `Array`, `ArrayBuffer`, `Set`, `Map`, even
`String`) shares the same set of methods. Learn them here once; they replace most of
the loops you would write in Python.

All examples use:

```scala
val lst = List(1, 2, 3, 5, 7, 10, 13)
val words = List("apple", "bob", "cherry", "date")
```

## 1. Transform: `map`, `flatMap`, `flatten`, `collect`

### `map`: change every element (same size out)

```scala
lst.map(x => x * 2)        // List(2, 4, 6, 10, 14, 20, 26)
lst.map(_ * 2)             // same, placeholder syntax
lst.map(x => "hi" * x)     // List(hi, hihi, hihihi, ...)
words.map(_.length)        // List(5, 3, 6, 4)
"hello".map(_.toUpper)     // "HELLO"
```

> **🐍** `[x * 2 for x in lst]` or `list(map(lambda x: x * 2, lst))`

### `flatten`: remove one level of nesting

```scala
List(List(1, 2, 3), List(3, 4, 5)).flatten   // List(1, 2, 3, 3, 4, 5)
List(Some(1), None, Some(3)).flatten         // List(1, 3)
```

### `flatMap`: `map`, then `flatten`

Use it when each element produces **zero or more** results.

```scala
lst.map(x => List(x, x + 1))
// List(List(1, 2), List(2, 3), List(3, 4), ...)  ← nested

lst.flatMap(x => List(x, x + 1))
// List(1, 2, 2, 3, 3, 4, 5, 6, 7, 8, 10, 11, 13, 14)  ← flat

words.flatMap(_.toList)            // all characters of all words
List("a b", "c d e").flatMap(_.split(" "))  // List(a, b, c, d, e)
```

```
lst.flatMap(x => List(x, x + 1))

step 1, map:      1 → List(1, 2)    2 → List(2, 3)   ...  13 → List(13, 14)
step 2, flatten:  List(1, 2, 2, 3, ..., 13, 14)
```

> **🐍** `[y for x in lst for y in (x, x + 1)]`

### `collect`: filter and map in one step, with pattern matching

```scala
val mixed: List[Any] = List(1, "two", 3, "four")
mixed.collect { case i: Int => i * 10 }      // List(10, 30)

lst.collect { case x if x % 2 == 0 => x * x } // List(4, 100)
```

## 2. Select: `filter`, `filterNot`, `partition`, `find`

```scala
lst.filter(_ % 2 != 0)        // List(1, 3, 5, 7, 13)   keep odd
lst.filterNot(_ % 2 != 0)     // List(2, 10)            drop odd
lst.partition(_ % 2 == 0)     // (List(2, 10), List(1, 3, 5, 7, 13))
lst.find(_ > 4)               // Some(5), first match (topic 15)
lst.find(_ > 100)             // None
```

> **🐍** `[x for x in lst if x % 2]`, `next((x for x in lst if x > 4), None)`

## 3. Test: `exists`, `forall`, `contains`, `count`

```scala
lst.exists(_ > 10)      // true   (Python: any(x > 10 for x in lst))
lst.forall(_ > 0)       // true   (Python: all(x > 0 for x in lst))
lst.contains(7)         // true   (Python: 7 in lst)
lst.count(_ % 2 == 0)   // 2      (Python: sum(1 for x in lst if x % 2 == 0))
```

## 4. Aggregate: `sum`, `max`, `reduce`, `foldLeft`

```scala
lst.sum                    // 41
lst.product                // 1 * 2 * 3 * 5 * 7 * 10 * 13 = 27300
lst.max / lst.min          // 13 / 1
words.maxBy(_.length)      // "cherry"
words.minBy(_.length)      // "bob"

lst.reduce(_ + _)          // 41, combines left to right
lst.foldLeft(0)(_ + _)     // 41, with a starting value
lst.foldLeft("")((acc, x) => acc + x)  // "123571013"
```

`foldLeft(start)(f)` walks the list keeping an accumulator:

```
foldLeft(0)(_ + _) on List(1, 2, 3)

acc = 0
acc = 0 + 1 = 1
acc = 1 + 2 = 3
acc = 3 + 3 = 6   → result 6
```

| | `reduce` | `foldLeft` |
|---|---|---|
| Start value | First element | You give one |
| Empty collection | **Throws** | Returns the start value |
| Result type | Same as elements | Anything (e.g. a `Map`) |

```scala
// foldLeft building a Map: count letters
"banana".foldLeft(Map.empty[Char, Int]) { (acc, ch) =>
  acc + (ch -> (acc.getOrElse(ch, 0) + 1))
}
// Map(b -> 1, a -> 3, n -> 2)
```

> **🐍** `functools.reduce(lambda acc, x: acc + x, lst, 0)`

`scanLeft` is `foldLeft` that keeps every intermediate result (running totals,
prefix sums):

```scala
lst.scanLeft(0)(_ + _)     // List(0, 1, 3, 6, 11, 18, 28, 41)
```

> **🐍** `list(itertools.accumulate(lst, initial=0))`

## 5. Slice: `take`, `drop`, `slice`, `splitAt`, `takeWhile`

```scala
lst.head / lst.last        // 1 / 13
lst.headOption             // Some(1), safe on empty
lst.tail / lst.init        // all but first / all but last
lst.take(3)                // List(1, 2, 3)          lst[:3]
lst.drop(3)                // List(5, 7, 10, 13)     lst[3:]
lst.takeRight(2)           // List(10, 13)           lst[-2:]
lst.dropRight(2)           // List(1, 2, 3, 5, 7)    lst[:-2]
lst.slice(2, 5)            // List(3, 5, 7)          lst[2:5]
lst.splitAt(3)             // (List(1, 2, 3), List(5, 7, 10, 13))
lst.takeWhile(_ < 5)       // List(1, 2, 3)          stop at first failure
lst.dropWhile(_ < 5)       // List(5, 7, 10, 13)
lst.span(_ < 5)            // (List(1, 2, 3), List(5, 7, 10, 13))
```

> **⚠️ Gotcha:** no negative indexes and no step. `lst[::2]` is
> `lst.zipWithIndex.collect { case (x, i) if i % 2 == 0 => x }` or
> `lst.grouped(2).map(_.head).toList`.

## 6. Sort: `sorted`, `sortBy`, `sortWith`, `reverse`

```scala
List(5, 3, 8).sorted                 // List(3, 5, 8)
List(5, 3, 8).sorted.reverse         // List(8, 5, 3)
List(5, 3, 8).sortWith(_ > _)        // List(8, 5, 3)
words.sortBy(_.length)               // List(bob, date, apple, cherry)
words.sortBy(w => (-w.length, w))    // by length desc, then alphabetically
List(5, 3, 8).sortBy(-_)             // descending for numbers
```

> **🐍** `sorted(words, key=len)` → `words.sortBy(_.length)`;
> `key=lambda w: (-len(w), w)` → `sortBy(w => (-w.length, w))`

## 7. Group: `groupBy`, `distinct`, `grouped`, `sliding`

```scala
words.groupBy(_.length)          // Map(5 -> List(apple), 3 -> List(bob), 6 -> List(cherry), 4 -> List(date))
words.groupBy(_.head)            // grouped by first letter
List(1, 1, 2, 3, 3).distinct     // List(1, 2, 3), keeps first-seen order
words.distinctBy(_.length)       // 2.13+ / Scala 3 only

List(1, 2, 3, 4, 5).grouped(2).toList   // List(List(1, 2), List(3, 4), List(5))  chunks
List(1, 2, 3, 4, 5).sliding(3).toList   // List(List(1, 2, 3), List(2, 3, 4), List(3, 4, 5))  windows
List(1, 2, 3, 4).sliding(2).map(w => w(1) - w(0)).toList  // List(1, 1, 1), differences
```

`grouped` and `sliding` return an **Iterator**; add `.toList` to see the values.

> **🐍** `grouped(n)` ≈ `[lst[i:i+n] for i in range(0, len(lst), n)]`;
> `sliding(n)` ≈ `[lst[i:i+n] for i in range(len(lst) - n + 1)]`

> **🔀 2 vs 3:** `groupMap`, `groupMapReduce`, `distinctBy` and `partitionMap` were
> added in 2.13 and are not in 2.12.

```scala
// 2.13+ / Scala 3: count words by first letter in one call
words.groupMapReduce(_.head)(_ => 1)(_ + _)    // Map(a -> 1, b -> 1, c -> 1, d -> 1)
```

## 8. Combine: `zip`, `zipWithIndex`, `unzip`, `++`

```scala
val names = List("Max", "Tom", "Ana")
val ages  = List(28, 35, 22)

names.zip(ages)          // List((Max,28), (Tom,35), (Ana,22))
names.zip(ages).toMap    // Map(Max -> 28, Tom -> 35, Ana -> 22)
names.zipWithIndex       // List((Max,0), (Tom,1), (Ana,2))
List((1, "a"), (2, "b")).unzip   // (List(1, 2), List(a, b))
names ++ List("Bo")      // List(Max, Tom, Ana, Bo)
```

> **🐍** `zip(a, b)`, `enumerate(xs)` (note: `zipWithIndex` puts the index **second**)

## 9. Index: `indexOf`, `indexWhere`, `indices`

```scala
lst.indexOf(7)             // 4   (-1 if missing)
lst.indexWhere(_ > 6)      // 4
lst.lastIndexOf(7)         // 4
lst.indices                // Range 0 until 7
for (i <- lst.indices) println(s"$i -> ${lst(i)}")
```

## 10. Iterate: `foreach`

```scala
lst.foreach(println)
lst.foreach(x => println(x * 2))
```

`foreach` returns `Unit`; it's for side effects (printing, updating a mutable
variable). If you're building a new collection, use `map` / `filter` instead.

## 11. Convert and display

```scala
lst.mkString(", ")           // "1, 2, 3, 5, 7, 10, 13"
lst.mkString("[", ", ", "]") // "[1, 2, 3, 5, 7, 10, 13]"
lst.toArray / lst.toVector / lst.toSet / lst.toBuffer
lst.reverse
lst.isEmpty / lst.nonEmpty / lst.size
```

## 12. Combinatorics and more

```scala
List(1, 2, 3).combinations(2).toList  // List(List(1, 2), List(1, 3), List(2, 3))
List(1, 2, 3).permutations.toList     // all 6 orderings
List(1, 2, 3).tails.toList            // List(List(1, 2, 3), List(2, 3), List(3), List())
List(1, 2, 3).inits.toList            // List(List(1, 2, 3), List(1, 2), List(1), List())
List.fill(3)("x")                     // List(x, x, x)
List.tabulate(3)(i => i * i)          // List(0, 1, 4)
List.iterate(1, 5)(_ * 2)             // List(1, 2, 4, 8, 16)
```

> **🐍** `itertools.combinations`, `itertools.permutations`

## Chaining

Operations chain naturally, reading left to right:

```scala
words
  .filter(_.length > 3)        // List(apple, cherry, date)
  .map(_.toUpperCase)          // List(APPLE, CHERRY, DATE)
  .sortBy(_.length)            // List(DATE, APPLE, CHERRY)
  .mkString(" | ")             // "DATE | APPLE | CHERRY"
```

The same chain as a `for` comprehension:

```scala
for (w <- words if w.length > 3) yield w.toUpperCase
```

### Lazy chaining with `view`

Each step above builds a full intermediate list. For long chains on big collections,
`.view` makes it lazy (like a Python generator):

```scala
(1 to 1000000).view.map(_ * 2).filter(_ % 3 == 0).take(5).toList
// List(6, 12, 18, 24, 30), only the needed elements are computed
```

## Operations on Maps

`map`, `filter` and friends see a `Map` as a collection of `(key, value)` tuples:

```scala
val mymap = Map(1 -> "Tom", 2 -> "Max", 3 -> "John")

mymap.map { case (k, v) => k -> ("hi " + v) }  // transform values
mymap.filter { case (k, _) => k > 1 }          // filter by key
mymap.map(_._2)                                // List(Tom, Max, John) (an Iterable)
mymap.view.mapValues(_.length).toMap           // 2.13+ / Scala 3
// mymap.mapValues(_.length)                   // 2.12 (deprecated later)
```

See [topic 13](13-maps.md) for more on maps.

## Python → Scala cheat sheet

| Python | Scala |
|--------|-------|
| `[f(x) for x in xs]` | `xs.map(f)` |
| `[x for x in xs if p(x)]` | `xs.filter(p)` |
| `[y for x in xs for y in g(x)]` | `xs.flatMap(g)` |
| `sum(xs)` / `max(xs)` | `xs.sum` / `xs.max` |
| `max(xs, key=f)` | `xs.maxBy(f)` |
| `sorted(xs, key=f)` | `xs.sortBy(f)` |
| `sorted(xs, reverse=True)` | `xs.sorted.reverse` / `xs.sortWith(_ > _)` |
| `any(p(x) for x in xs)` | `xs.exists(p)` |
| `all(p(x) for x in xs)` | `xs.forall(p)` |
| `len([x for x in xs if p(x)])` | `xs.count(p)` |
| `functools.reduce(f, xs, init)` | `xs.foldLeft(init)(f)` |
| `itertools.accumulate(xs)` | `xs.scanLeft(0)(_ + _).tail` |
| `enumerate(xs)` | `xs.zipWithIndex` (index second!) |
| `zip(a, b)` | `a.zip(b)` |
| `dict(zip(a, b))` | `a.zip(b).toMap` |
| `list(set(xs))` (keeping order) | `xs.distinct` |
| `Counter(xs)` | `xs.groupBy(identity).map { case (k, v) => k -> v.size }` |
| `itertools.groupby` / `defaultdict(list)` | `xs.groupBy(f)` |
| `xs[a:b]` | `xs.slice(a, b)` |
| `xs[::-1]` | `xs.reverse` |
| `", ".join(xs)` | `xs.mkString(", ")` |
| `next(x for x in xs if p(x))` | `xs.find(p)` (returns `Option`) |

> **🎯 Interview one-liner**
> "`map` transforms every element and returns a collection of the same size. `flatMap`
> turns each element into a collection and flattens the results, for when one element
> produces several outputs. `filter` keeps the elements that match. `foldLeft` reduces a
> collection to one value with an accumulator."

## Practice

1. From `List("a b c", "d e")`, get `List("A", "B", "C", "D", "E")` in one chain.
2. Find the longest word in a sentence with `maxBy`.
3. Compute running totals of `List(3, 1, 4, 1, 5)` with `scanLeft`.
4. Find all pairs in `List(1, 2, 3, 4)` that sum to 5 using `combinations`.

---
Previous: [13. Maps](13-maps.md) · Next: [15. Option](15-option.md)
