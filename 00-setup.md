# 00. Setup and Tooling

## What you need

| Python world | Scala world | What it does |
|--------------|-------------|--------------|
| CPython interpreter | **JDK** (Java 11, 17 or 21) | Scala compiles to JVM bytecode, so it needs Java installed |
| `python` REPL | `scala` / `scala-cli repl` | Interactive shell |
| `pip` + `venv` + `pyproject.toml` | **sbt** + `build.sbt` | Dependencies and builds |
| Running a single `.py` file | **scala-cli** | Run a single `.scala` file with no project setup |

## Install

The easiest route is **Coursier** (`cs`), the official Scala installer:

```bash
# macOS
brew install coursier && coursier setup

# Linux
curl -fL https://github.com/coursier/coursier/releases/latest/download/cs-x86_64-pc-linux.gz | gzip -d > cs
chmod +x cs && ./cs setup
```

`cs setup` installs a JDK (if missing), `scala`, `scala-cli` and `sbt`.

## Your first program

Create `hello.scala`:

```scala
// Scala 3
@main def hello(): Unit =
  println("Hello, Scala!")
```

```scala
// Scala 2.12 / 2.13
object Hello {
  def main(args: Array[String]): Unit = {
    println("Hello, Scala!")
  }
}
```

Run it:

```bash
scala-cli run hello.scala                 # uses Scala 3 by default
scala-cli run hello.scala --scala 2.12    # pick a version
```

> **🐍 Python equivalent**
> ```python
> if __name__ == "__main__":
>     print("Hello, Scala!")
> ```

> **🔀 2 vs 3**
> Scala 2 needs an `object` with a `main(args: Array[String])` method.
> Scala 3 adds `@main`, which can sit on any top-level function.

## The REPL

```bash
scala-cli repl                 # Scala 3
scala-cli repl --scala 2.13    # Scala 2
```

```scala
scala> 1 + 1
val res0: Int = 2
```

The REPL prints both the **value** and its **type**. Use it to check what type
Scala infers for an expression.

## Compiled, not interpreted

> **⚠️ Gotcha**
> Python finds most errors only when a line runs. Scala's compiler checks types
> **before** anything runs. Your first weeks will feel like arguing with the
> compiler; that is normal, and it catches bugs that Python would hit in production.

## Editor setup

- **VS Code + Metals** extension (free, works well for beginners)
- **IntelliJ IDEA + Scala plugin**

Both give you autocompletion, type-on-hover and inline errors.

---
Next: [01. Values, variables and types](01-values-and-types.md)
