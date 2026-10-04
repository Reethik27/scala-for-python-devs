# 18. DSA Patterns in Scala (LeetCode Style)

How the common interview patterns look in Scala, for someone used to solving them
in Python. Each solution is written in the "mutable, imperative" style that maps
directly from Python, and some also show the idiomatic functional version.

## LeetCode setup

LeetCode wraps Scala solutions in an `object Solution`:

```scala
object Solution {
  def twoSum(nums: Array[Int], target: Int): Array[Int] = {
    ...
  }
}
```

Useful imports at the top:

```scala
import scala.collection.mutable
// then: mutable.Map, mutable.Set, mutable.ArrayBuffer, mutable.Queue, mutable.Stack ...
```

## Python → Scala data structures for DSA

| Python | Scala |
|--------|-------|
| `d = {}` | `val d = mutable.Map[Int, Int]()` |
| `defaultdict(int)` | `mutable.Map[Char, Int]().withDefaultValue(0)` |
| `defaultdict(list)` | `mutable.Map[String, mutable.ListBuffer[String]]()` + `getOrElseUpdate` |
| `Counter(s)` | `s.groupBy(identity).map { case (k, v) => k -> v.length }` |
| `s = set()` | `val s = mutable.Set[Int]()` |
| `res = []` | `val res = mutable.ArrayBuffer[Int]()` |
| `stack = []` | `val stack = mutable.Stack[Int]()` (`push` / `pop` / `top`) |
| `deque()` | `val q = mutable.Queue[Int]()` (`enqueue` / `dequeue`) |
| `heapq` (min-heap) | `mutable.PriorityQueue` with a reversed ordering (see section 10) |
| `float('inf')` | `Int.MaxValue` / `Double.PositiveInfinity` |
| `for i in range(n)` | `for (i <- 0 until n)` |
| `for i, x in enumerate(nums)` | `for ((x, i) <- nums.zipWithIndex)` |
| `max(a, b)` | `math.max(a, b)` or `a max b` |
| `return` early | Fine inside a `while` loop; inside `for` see the ⚠️ below |

## 1. Two Sum (hash map)

The notes had the right idea, but `map` and `d` weren't declared consistently.
Fixed and complete:

```scala
import scala.collection.mutable

object Solution {
  def twoSum(nums: Array[Int], target: Int): Array[Int] = {
    val seen = mutable.Map[Int, Int]()          // value -> index
    var i = 0
    while (i < nums.length) {
      val complement = target - nums(i)
      if (seen.contains(complement)) return Array(seen(complement), i)
      seen(nums(i)) = i
      i += 1
    }
    Array()                                     // no answer found
  }
}

Solution.twoSum(Array(2, 7, 11, 15), 9)   // Array(0, 1)
```

> **⚠️ Gotcha: `return` inside a `for` loop**
> The notes used `for (i <- nums.indices) { ... return ... }`. That works on Scala 2,
> but a `for` loop is really a lambda passed to `foreach`, so `return` has to jump out
> of the lambda (a "non-local return", implemented with an exception).
> **Scala 3 deprecates this and warns.** Two fixes:
> - Use a `while` loop, as above. `return` inside `while` is an ordinary return, in every version.
> - In Scala 3, use `boundary` / `break`:
>   ```scala
>   import scala.util.boundary, boundary.break
>
>   def twoSum(nums: Array[Int], target: Int): Array[Int] =
>     boundary:
>       val seen = mutable.Map[Int, Int]()
>       for i <- nums.indices do
>         val complement = target - nums(i)
>         if seen.contains(complement) then break(Array(seen(complement), i))
>         seen(nums(i)) = i
>       Array()
>   ```

> **🐍 Python version**
> ```python
> seen = {}
> for i, x in enumerate(nums):
>     if target - x in seen: return [seen[target - x], i]
>     seen[x] = i
> ```

## 2. Valid Anagram (counting with a map)

```scala
import scala.collection.mutable

object Solution {
  def isAnagram(s: String, t: String): Boolean = {
    if (s.length != t.length) return false
    val counts = mutable.Map[Char, Int]().withDefaultValue(0)
    for (ch <- s) counts(ch) += 1
    for (ch <- t) counts(ch) -= 1
    counts.values.forall(_ == 0)
  }
}
```

Idiomatic one-liners:

```scala
def isAnagram(s: String, t: String): Boolean = s.sorted == t.sorted
def isAnagram2(s: String, t: String): Boolean = s.groupBy(identity) == t.groupBy(identity)
```

> **⚠️ Gotcha from the notes:** `countS = scala.collection.mutable.Map[Char, Int]()`
> needs `val` in front: `val countS = mutable.Map[Char, Int]()`. And `d = {}` is Python;
> `{}` in Scala is an empty block that returns `Unit`.

## 3. Group Anagrams (map of lists)

The version in the notes works. Here it is with `getOrElseUpdate`, which replaces the
`if contains ... else ...` check:

```scala
import scala.collection.mutable

object Solution {
  def groupAnagrams(strs: Array[String]): List[List[String]] = {
    val groups = mutable.Map[String, mutable.ListBuffer[String]]()
    for (str <- strs) {
      val key = str.sorted                              // "eat" -> "aet"
      groups.getOrElseUpdate(key, mutable.ListBuffer()) += str
    }
    groups.values.map(_.toList).toList
  }
}
```

Idiomatic one-liner:

```scala
def groupAnagrams(strs: Array[String]): List[List[String]] =
  strs.groupBy(_.sorted).values.map(_.toList).toList
```

## 4. Longest Substring Without Repeating Characters (sliding window)

The draft in the notes mixed Python and Scala and wouldn't compile:

```scala
val charSet = scala.collection.mutable.set[Int]()   // ✗ `Set` needs a capital S; should hold Char
var s = String()                                     // ✗ not valid, and unused
while(nums(r) in charSet){                           // ✗ `in` is Python; use charSet.contains(...)
  charset                                            // ✗ wrong case, and nothing is removed
}
longest = math.max(longest, len(charset))            // ✗ `len` is Python; window size is r - l + 1
```

Corrected:

```scala
import scala.collection.mutable

object Solution {
  def lengthOfLongestSubstring(s: String): Int = {
    val window = mutable.Set[Char]()
    var l = 0
    var longest = 0

    for (r <- s.indices) {
      while (window.contains(s(r))) {   // shrink until s(r) is no longer a duplicate
        window -= s(l)
        l += 1
      }
      window += s(r)
      longest = math.max(longest, r - l + 1)
    }
    longest
  }
}

Solution.lengthOfLongestSubstring("abcabcbb")   // 3 ("abc")
Solution.lengthOfLongestSubstring("bbbbb")      // 1
Solution.lengthOfLongestSubstring("pwwkew")     // 3 ("wke")
```

> **🐍 Python version**
> ```python
> window, l, longest = set(), 0, 0
> for r in range(len(s)):
>     while s[r] in window:
>         window.remove(s[l]); l += 1
>     window.add(s[r])
>     longest = max(longest, r - l + 1)
> ```

## 5. Contains Duplicate (set)

```scala
def containsDuplicate(nums: Array[Int]): Boolean = {
  val seen = mutable.Set[Int]()
  nums.exists(x => !seen.add(x))     // add returns false if x was already there
}

// or simply
def containsDuplicate2(nums: Array[Int]): Boolean = nums.distinct.length != nums.length
```

## 6. Two pointers (sorted array)

```scala
def twoSumSorted(numbers: Array[Int], target: Int): Array[Int] = {
  var l = 0
  var r = numbers.length - 1
  while (l < r) {
    val sum = numbers(l) + numbers(r)
    if (sum == target) return Array(l + 1, r + 1)
    else if (sum < target) l += 1
    else r -= 1
  }
  Array()
}
```

## 7. Valid Parentheses (stack)

```scala
def isValid(s: String): Boolean = {
  val stack = mutable.Stack[Char]()
  val pairs = Map(')' -> '(', ']' -> '[', '}' -> '{')
  var i = 0
  while (i < s.length) {               // while, so `return` is safe in Scala 3 too
    val ch = s(i)
    if (pairs.contains(ch)) {
      if (stack.isEmpty || stack.pop() != pairs(ch)) return false
    } else stack.push(ch)
    i += 1
  }
  stack.isEmpty
}
```

## 8. Top K Frequent Elements (counting + sorting)

```scala
def topKFrequent(nums: Array[Int], k: Int): Array[Int] =
  nums.groupBy(identity)
    .map { case (n, occurrences) => n -> occurrences.length }
    .toArray
    .sortBy { case (_, count) => -count }
    .take(k)
    .map(_._1)
```

## 9. BFS (queue)

```scala
def bfs(graph: Map[Int, List[Int]], start: Int): List[Int] = {
  val visited = mutable.Set(start)
  val queue   = mutable.Queue(start)
  val order   = mutable.ListBuffer[Int]()
  while (queue.nonEmpty) {
    val node = queue.dequeue()
    order += node
    for (next <- graph.getOrElse(node, Nil) if visited.add(next)) queue.enqueue(next)
  }
  order.toList
}
```

## 10. Min-heap (priority queue)

Scala's `PriorityQueue` is a **max**-heap by default (Python's `heapq` is a min-heap):

```scala
val maxHeap = mutable.PriorityQueue(3, 1, 4)
maxHeap.dequeue()                         // 4

// Scala 2.12 / 2.13
val minHeap = mutable.PriorityQueue[Int]()(Ordering[Int].reverse)
// Scala 3
val minHeap = mutable.PriorityQueue.empty[Int](using Ordering[Int].reverse)

minHeap.enqueue(3, 1, 4)
minHeap.dequeue()                         // 1
minHeap.head                              // 3 (peek)
```

> **⚠️ Gotcha:** printing or iterating a `PriorityQueue` does **not** show elements in
> sorted order. Only `dequeue()` / `head` respect the ordering.

## Common Scala mistakes in LeetCode

| Mistake | Fix |
|---------|-----|
| `x = mutable.Map()` | `val x = mutable.Map[K, V]()` (types needed for empty collections) |
| `arr[i]` | `arr(i)` |
| `len(x)` | `x.length` / `x.size` |
| `x in s` | `s.contains(x)` |
| `for i in range(n)` | `for (i <- 0 until n)` |
| `i++` | `i += 1` |
| `return` inside `for` or a lambda | Use a `while` loop, `exists`/`find`, or Scala 3 `boundary` |
| `Int` overflow on big sums | Use `Long` |

---
Previous: [15. Option](15-option.md) · Next: 19. Generics (coming soon)
