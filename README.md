# Scala for Python Developers

Beginner-friendly Scala notes, written for someone who already knows Python well.
Each topic lives in its own file. Every topic covers **Scala 2.12/2.13 and Scala 3**,
and calls out where they differ.

## How to read these docs

- Read the topics in order; each one builds on the last.
- Look for these boxes:
  - **🐍 Python equivalent** shows the closest Python code.
  - **🔀 2 vs 3** marks syntax or behaviour that changed between Scala 2 and Scala 3.
  - **⚠️ Gotcha** marks something that surprises Python developers.
  - **🎯 Interview one-liner** is a sentence to remember for interviews.
- Try every example yourself in the REPL (see topic 00).

## Outline

### Part 1: Foundations
| # | Topic | Status |
|---|-------|--------|
| 00 | [Setup and tooling](00-setup.md) (JDK, scala-cli, sbt, REPL) | ✅ |
| 01 | [Values, variables and types](01-values-and-types.md) | ✅ |
| 02 | [Control flow as expressions](02-control-flow.md) (`if`, `match`, loops, `for`) | ✅ |
| 03 | [Functions, closures and currying](03-functions.md) | ✅ |
| 04 | [Strings and string interpolation](04-strings.md) | ✅ |

### Part 2: Organising code
| # | Topic | Status |
|---|-------|--------|
| 05 | [Classes, objects and companion objects](05-classes-and-objects.md) | ✅ |
| 06 | Case classes (Scala's dataclasses) | Planned |
| 07 | Traits (interfaces and mixins) | Planned |
| 08 | Packages, imports and the entry point (`main`) | Planned |

### Part 3: Collections and working with data
| # | Topic | Status |
|---|-------|--------|
| 09 | [Collections overview: which one to use](09-collections-overview.md) | ✅ |
| 10 | [Arrays and ArrayBuffer](10-arrays.md) | ✅ |
| 11 | [Lists and ListBuffer](11-lists.md) | ✅ |
| 12 | [Sets](12-sets.md) | ✅ |
| 13 | [Maps](13-maps.md) | ✅ |
| 14 | [Collection operations: map, flatMap, fold, groupBy, sliding...](14-collection-operations.md) | ✅ |
| 15 | [Option: values that might be missing](15-option.md) | ✅ |
| 16 | Error handling: `Try`, `Either`, exceptions | Planned |
| 17 | Pattern matching in depth | Planned |
| 18 | [DSA patterns in Scala (LeetCode style)](18-dsa-patterns.md) | ✅ |

### Part 4: The type system
| # | Topic | Status |
|---|-------|--------|
| 19 | Generics and variance (`+A`, `-A`) | Planned |
| 20 | Enums and ADTs (`sealed trait` vs Scala 3 `enum`) | Planned |
| 21 | Implicits (Scala 2) vs `given`/`using` (Scala 3) | Planned |
| 22 | Extension methods and type classes | Planned |

### Part 5: Real-world Scala
| # | Topic | Status |
|---|-------|--------|
| 23 | Concurrency with `Future` | Planned |
| 24 | Building projects with sbt | Planned |
| 25 | Testing (ScalaTest / MUnit) | Planned |
| 26 | Migrating from Scala 2.12 to Scala 3 (checklist) | Planned |
| 27 | Python ↔ Scala cheat sheet | Planned |

## Version quick reference

| Version | Released | Notes |
|---------|----------|-------|
| 2.12 | 2016 | Common in older Spark and enterprise codebases |
| 2.13 | 2019 | Redesigned collections library (Scala 3 uses it too) |
| 3.x | 2021+ | New syntax (optional braces), `enum`, `given`/`using`, union types |

Scala 3 can use most Scala 2.13 libraries, so mixed codebases are common during migration.

## About the examples

The main examples in topics 03 to 05 and 09 to 18 were compiled and run on Scala 2.12, 2.13 and 3, and the outputs in the comments come from those runs.
Where a line only works on some versions, the comment next to it says so.
