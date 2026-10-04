# 04. Strings

A Scala `String` is a Java `java.lang.String`, plus extra Scala methods that make it
behave like a collection of `Char`s. Strings are **immutable**: every method returns a
new string.

## Creating strings

```scala
val str1: String = "Hello World"
val str2 = "Max"                 // type inferred
val c: Char = 'H'                // single quotes = one Char, not a String
```

Multi-line strings use triple quotes, like Python:

```scala
val poem =
  """Roses are red,
    |Violets are blue""".stripMargin   // stripMargin removes everything up to the |
```

## Joining strings

```scala
str1 + str2          // "Hello WorldMax"
str1.concat(str2)    // "Hello WorldMax" (same thing, Java method)
"ab" * 3             // "ababab", same as Python
```

## String interpolation (the preferred way)

Put a letter in front of the quotes:

```scala
val name = "Max"
val age  = 25

s"My name is $name and I am $age years old"   // s: plain substitution
s"Next year I will be ${age + 1}"             // ${...} for any expression

val price = 100.25
f"Price: $price%.2f, age: $age%d"             // f: printf-style, type-checked
f"$name%-10s|"                                // "Max       |" (left-aligned, width 10)

raw"Hello\nWorld"                             // raw: \n stays as two characters
```

> **🐍 Python equivalent**
> | Scala | Python |
> |-------|--------|
> | `s"Hi $name"` | `f"Hi {name}"` |
> | `s"${age + 1}"` | `f"{age + 1}"` |
> | `f"$price%.2f"` | `f"{price:.2f}"` |
> | `raw"a\nb"` | `r"a\nb"` |

> **⚠️ Gotcha**
> Python's `f"..."` is Scala's **`s"..."`**. Scala's `f"..."` is for format specifiers
> like `%.2f`, and the compiler checks that `%d` gets an `Int`.

## `println`, `printf` and `.format`

```scala
val num1 = 75
val num2 = 100.25

println(str1)                                            // prints, then a newline

printf("(%d -- %f -- %s)\n", num1, num2, str1)           // formats AND prints
// (75 -- 100.250000 -- Hello World)

val result = "(%d -- %f -- %s)".format(num1, num2, str1) // formats, RETURNS a String
println(result)
// (75 -- 100.250000 -- Hello World)
```

> **⚠️ Gotcha**
> `printf` returns `Unit`, not a String. `val r = printf(...)` prints the text and
> then `r` is `()`. Use `.format` or `f"..."` when you need the string.
> Also note that `printf` does not add a newline; add `\n` yourself.

## Common methods

```scala
val str = "Hello World"

str.length                     // 11 (str.length() also works)
str.toUpperCase                // "HELLO WORLD"
str.toLowerCase                // "hello world"
"  hi  ".trim                  // "hi"
str.replace("World", "Scala")  // "Hello Scala"
str.contains("World")          // true
str.startsWith("Hello")        // true
str.endsWith("World")          // true
str.substring(0, 5)            // "Hello" (end index excluded, like str[0:5])
str.split(" ")                 // Array("Hello", "World")
str.isEmpty                    // false
str.nonEmpty                   // true
str.reverse                    // "dlroW olleH"
str.charAt(0)                  // 'H'
str(0)                         // 'H' (Scala shortcut)
str.indexOf("World")           // 6
str.indexOf("xyz")             // -1 (not found, like Python's str.find)
```

## Strings as collections

Scala treats a `String` like a sequence of `Char`, so the collection methods from
topic 14 work on it:

```scala
"hello".map(_.toUpper)         // "HELLO"
"hello".filter(_ != 'l')       // "heo"
"hello".count(_ == 'l')        // 2
"hello".toList                 // List('h', 'e', 'l', 'l', 'o')
"hello".sorted                 // "ehllo"   ← handy for anagram problems
"hello".distinct               // "helo"
"hello".take(3)                // "hel"
"hello".drop(3)                // "lo"
"hello".takeRight(2)           // "lo"
"hello".headOption             // Some('h')
"a,b,,c".split(",")            // Array("a", "b", "", "c")
"hello".groupBy(identity)      // Map(e -> "e", l -> "ll", h -> "h", o -> "o"), order not guaranteed
for (ch <- "abc") println(ch)  // loop over characters
```

## Python slicing cheat sheet

| Python | Scala |
|--------|-------|
| `s[0]` | `s(0)` or `s.charAt(0)` |
| `s[-1]` | `s.last` or `s(s.length - 1)` |
| `s[1:4]` | `s.substring(1, 4)` or `s.slice(1, 4)` |
| `s[:3]` | `s.take(3)` |
| `s[3:]` | `s.drop(3)` |
| `s[::-1]` | `s.reverse` |
| `len(s)` | `s.length` |
| `"x" in s` | `s.contains("x")` |
| `s.strip()` | `s.trim` (`s.strip` on Java 11+) |
| `", ".join(lst)` | `lst.mkString(", ")` |
| `int("42")` | `"42".toInt` |
| `str(42)` | `42.toString` |

## Converting

```scala
"42".toInt          // 42 (throws NumberFormatException on bad input)
"42".toIntOption    // Some(42); "4x".toIntOption is None (2.13+ and Scala 3)
"3.14".toDouble     // 3.14
42.toString         // "42"
'a'.toInt           // 97 (character code, like Python's ord)
97.toChar           // 'a' (like Python's chr)
'7'.asDigit         // 7
```

> **🔀 2 vs 3**
> `toIntOption`, `toDoubleOption` and friends were added in Scala **2.13**.
> On 2.12, use `scala.util.Try("42".toInt).toOption`.

## Practice

1. Check whether a string is a palindrome, ignoring case.
2. Count the vowels in a string with `count`.
3. Print a formatted table row with `f"$name%-10s$score%5.1f"`.

> **🎯 Interview one-liner**
> "Strings in Scala are immutable Java strings with extra collection methods. Prefer
> `s` and `f` interpolators over `+` concatenation and `printf`."

---
Previous: [03. Functions](03-functions.md) · Next: [05. Classes and objects](05-classes-and-objects.md)
