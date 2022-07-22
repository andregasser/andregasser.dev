---
title: What's New in Kotlin 1.7.0
date: 2022-06-22T08:24:01+02:00
summary: A new release of Kotlin has been made available on 9 June 2022. Let's dive in a bit and find ou what new features and improvements it brings us as developers.
description: Article description.
draft: false 
featured: true
featureImage: /images/kotlin/kotlin-logo-with-text.png
thumbnail: /images/kotlin/kotlin-logo.png
categories:
  - Software Engineering
tags:
  - Kotlin
---

Kotlin 1.7.0 has been released on 9 June 2022 and with it comes a big list of improvements/new features along the way. In this article, we will have a closer look at the features offered to Kotlin developers working on the JVM. There were many more changes of course, such as in Kotlin/JS or Kotlin/Native. For the full list of changes checkout [this page](https://kotlinlang.org/docs/whatsnew17.html).

# Kotlin/K2 Compiler
{{% notice note "Note" %}}
This feature is still experimental.
{{% /notice %}}

The new K2 compiler for Kotlin brings a massive performance boost to your project. You can expect a performance boost of factor x2 and more compared to the Kotlin/JVM compiler. That table speaks for itself:

| Project       | Current Kotlin Compiler Performance | New K2 Kotlin Compiler Performance | Performance Boost |
| :-----------: | ----------------------------------: | ---------------------------------: | ----------------: |
| Kotlin        | 2.2 KLOC/s                          | 4.8 KLOC/s                         | ~ x2.2            |
| YouTrack      | 1.8 KLOC/s                          | 4.2 KLOC/s                         | ~ x2.3            |
| IntelliJ IDEA | 1.8 KLOC/s                          | 3.9 KLOC/s                         | ~ x2.2            |
| Space         | 1.2 KLOC/s                          | 2.8 KLOC/s                         | ~ x2.3            |

{{% notice info "Info" %}}
KLOC/s stands for the number of thousands of lines of code that the compiler processes per second.
{{% /notice %}}

The compiler is not feature complete and the Kotlin team is working on stabilizing it during the next releases. If you want to enable the K2 compiler for your project, use the following compiler option:

```bash
-Xuse-k2
```

# Kotlin/JVM Compiler
The Kotlin team has also worked on performance improvements for the current stable compiler, the Kotlin/JVM compiler. Compared to Kotlin 1.6.0, compilation time has been reduced by 10% on average.

## New Underscore Operator
On the language side of Kotlin, we get a new underscore operator, `_`, for type arguments. It can be used to automatically infer the type of a type argument when other types are specified. Look at the code below (especially the main function) and you'll understand how it works:

```kotlin
abstract class SomeClass<T> {
    abstract fun execute(): T
}

class SomeImplementation : SomeClass<String>() {
    override fun execute(): String = "Test"
}

class OtherImplementation : SomeClass<Int>() {
    override fun execute(): Int = 42
}

object Runner {
    inline fun <reified S: SomeClass<T>, T> run(): T {
        return S::class.java.getDeclaredConstructor().newInstance().execute()
    }
}

fun main() {
    // T is inferred as String because SomeImplementation derives from SomeClass<String>
    val s = Runner.run<SomeImplementation, _>()
    assert(s == "Test")

    // T is inferred as Int because OtherImplementation derives from SomeClass<Int>
    val n = Runner.run<OtherImplementation, _>()
    assert(n == 42)
}
```

# Standard Library

## Deep Recursive Functions
There was also a nice addition to the Standard Library of Kotlin. The feature is called **Deep Recursive Functions**. It is useful for cases, where deep recursion is required. It allows you to define a function which keeps its stack on the heap and thus does not throw a StackOverflowError when it calls itself recursively 100'000 times. The Kotlin team recommends to use this feature if recursion depth exceeds 1000 calls. 

Have a look at this code, which generates a tree consisting of 100'000 nodes:

```kotlin
class Tree(val left: Tree? = null, val right: Tree? = null)
val deepTree = generateSequence(Tree()) { Tree(it) }.take(100_000).last()
```

And here is an implementation of a function that calculates the depth of the tree using the regular, recursive approach. It will throw a StackOverflowError:

```kotlin
fun depth(t: Tree?): Int =
    if (t == null) 0 else max(depth(t.left), depth(t.right)) + 1
println(depth(deepTree)) // StackOverflowError
```

In this second example, the same recursion is implemented using `DeepRecursiveFunction`:

```kotlin
val depth = DeepRecursiveFunction<Tree?, Int> { t ->
    if (t == null) 0 else max(callRecursive(t.left), callRecursive(t.right)) + 1
}
println(depth(deepTree)) // Ok
```

This is cool stuff and if you're interested in more detail, you can find more information [here](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-deep-recursive-function/).

## New Extension Functions for Java Optionals
{{% notice note "Note" %}}
This feature is still experimental.
{{% /notice %}}

The new Kotlin release brings us new extension functions for the Java Optional type, such as `getOrNull()`, `getOrDefault()` and `getOrElse()`. This comes really handy, as they either return the value inside the Optional if present, or `null, the types default value or something else specified by the developer.

Here's some example code that shows these functions in action:

```kotlin
val presentOptional = Optional.of("I'm here!")

println(presentOptional.getOrNull())
// "I'm here!"

val absentOptional = Optional.empty<String>()

println(absentOptional.getOrNull())
// null

println(absentOptional.getOrDefault("Nobody here!"))  
// "Nobody here!"

println(absentOptional.getOrElse {
    println("Optional was absent!")
    "Default value!"
})
// "Optional was absent!"
// "Default value!"
```

# Conclusion
Besides massive performance optimizations, the new Kotlin release has also brought some really nice feature additions. 

