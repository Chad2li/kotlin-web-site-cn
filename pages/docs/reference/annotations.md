---
type: doc
layout: reference
category: "Syntax"
title: "注解"
---

# 注解

## 注解声明
注解是将元数据附加到代码的方法。要声明注解，请将 *annotation*{: .keyword } 修饰符放在类的前面：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
annotation class Fancy
```
</div>

注解的附加属性可以通过用元注解标注注解类来指定：

  * [`@Target`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) 指定可以用<!--
    -->该注解标注的元素的可能的类型（类、函数、属性、表达式等）；
  * [`@Retention`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) 指定该注解是否<!--
    -->存储在编译后的 class 文件中，以及它在运行时能否通过反射可见
    （默认都是 true）；
  * [`@Repeatable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) 允许<!--
    -->在单个元素上多次使用相同的该注解；
  * [`@MustBeDocumented`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) 指定<!--
    -->该注解是公有 API 的一部分，并且应该包含在<!--
    -->生成的 API 文档中显示的类或方法的签名中。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```
</div>

### 用法

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```
</div>

如果需要对类的主构造函数进行标注，则需要在构造函数声明中添加 *constructor*{: .keyword} 关键字
，并将注解添加到其前面：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Foo @Inject constructor(dependency: MyDependency) { …… }
```
</div>

你也可以标注属性访问器：

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```
</div>

### 构造函数

注解可以有接受参数的构造函数。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```
</div>

允许的参数类型有：

 * 对应于 Java 原生类型的类型（Int、 Long等）；
 * 字符串；
 * 类（`Foo::class`）；
 * 枚举；
 * 其他注解；
 * 上面已列类型的数组。

注解参数不能有可空类型，因为 JVM 不支持将 `null` 作为<!--
-->注解属性的值存储。

如果注解用作另一个注解的参数，则其名称不以 @ 字符为前缀：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```
</div>

如果需要将一个类指定为注解的参数，请使用 Kotlin 类
（[KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)）。Kotlin 编译器会<!--
-->自动将其转换为 Java 类，以便 Java 代码能够正常访问该注解与参数
。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin

import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any>)

@Ann(String::class, Int::class) class MyClass
```
</div>

### Lambda 表达式

注解也可以用于 lambda 表达式。它们会被应用于生成 lambda 表达式体的 `invoke()`
方法上。这对于像 [Quasar](http://www.paralleluniverse.co/quasar/) 这样的框架很有用，
该框架使用注解进行并发控制。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```
</div>

## 注解使用处目标

当对属性或主构造函数参数进行标注时，从相应的 Kotlin 元素<!--
-->生成的 Java 元素会有多个，因此在生成的 Java 字节码中该注解有多个可能位置
。如果要指定精确地指定应该如何生成该注解，请使用以下语法：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Example(@field:Ann val foo,    // 标注 Java 字段
              @get:Ann val bar,      // 标注 Java getter
              @param:Ann val quux)   // 标注 Java 构造函数参数
```
</div>

可以使用相同的语法来标注整个文件。 要做到这一点，把带有目标 `file` 的注解放在<!--
-->文件的顶层、package 指令之前或者在所有导入之前（如果文件在默认包中的话）：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```
</div>

如果你对同一目标有多个注解，那么可以这样来避免目标重复——在目标后面添加方括号<!--
-->并将所有注解放在方括号内：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Example {
     @set:[Inject VisibleForTesting]
     var collaborator: Collaborator
}
```
</div>

支持的使用处目标的完整列表为：

  * `file`；
  * `property`（具有此目标的注解对 Java 不可见）；
  * `field`；
  * `get`（属性 getter）；
  * `set`（属性 setter）；
  * `receiver`（扩展函数或属性的接收者参数）；
  * `param`（构造函数参数）；
  * `setparam`（属性 setter 参数）；
  * `delegate`（为委托属性存储其委托实例的字段）。

要标注扩展函数的接收者参数，请使用以下语法：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun @receiver:Fancy String.myExtension() { ... }
```
</div>

如果不指定使用处目标，则根据正在使用的注解的 `@Target` 注解来选择目标
。如果有多个适用的目标，则使用以下列表中的第一个适用目标：

  * `param`;
  * `property`;
  * `field`.

## Java 注解

Java 注解与 Kotlin 100% 兼容：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    // 将 @Rule 注解应用于属性 getter
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```
</div>

因为 Java 编写的注解没有定义参数顺序，所以不能使用常规函数调用<!--
-->语法来传递参数。相反，你需要使用具名参数语法：

<div class="sample" markdown="1" mode="java" theme="idea">

``` java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```
</div>

就像在 Java 中一样，一个特殊的情况是 `value` 参数；它的值无需显式名称指定：

<div class="sample" markdown="1" theme="idea" mode="java">

``` java
// Java
public @interface AnnWithValue {
    String value();
}
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Kotlin
@AnnWithValue("abc") class C
```
</div>

### 数组作为注解参数

如果 Java 中的 `value` 参数具有数组类型，它会成为 Kotlin 中的一个 `vararg` 参数：

<div class="sample" markdown="1" theme="idea" mode="java">

``` java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```
</div>

对于具有数组类型的其他参数，你需要显式使用数组字面值语法（自 Kotlin 1.2 起）或者
`arrayOf(……)`：

<div class="sample" markdown="1" theme="idea" mode="java">

``` java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Kotlin 1.2+：
@AnnWithArrayMethod(names = ["abc", "foo", "bar"]) 
class C

// 旧版本 Kotlin：
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) 
class D
```
</div>

### 访问注解实例的属性

注解实例的值会作为属性暴露给 Kotlin 代码：

<div class="sample" markdown="1" theme="idea" mode="java">

``` java
// Java
public @interface Ann {
    int value();
}
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```
</div>
