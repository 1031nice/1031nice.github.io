---
layout: post
title:  Kotlin for Java Developers - Extension functions
date:   2022-04-08 23:13 +0900
categories: TIL
---

### TOC

[1. Extension functions](#1.-Extension-functions)
<br>[2. 표준 라이브러리 속 확장 함수](#2.-표준-라이브러리-속-확장-함수)
<br>[3. Calling Extensions](#3.-Calling-Extensions)
<br>[4. 확장 함수가 중요한 이유](#4.-확장-함수가-중요한-이유)
<br>[5. TMI](#5.-TMI)

### 1. Extension functions

**<u>Extension functions</u>**

Extension functions(이하 확장 함수)는 클래스 외부에서 정의되지만 클래스의 멤버 함수인 것처럼 사용할 수 있는 함수이다.

```kotlin
// 확장 함수 정의
fun String.lastChar() = this.get(this.length - 1)

// 확장 함수 사용하는 예시
val c: Char = "abc".lastChar()
```
위와 같이 `lastChar()`라는 extension 함수를 정의하면
* `lastChar` 함수가 마치 String 클래스에 정의되어 있는 것처럼 사용할 수 있다.
* 확장 함수는 클래스를 확장하는데, 확장 함수가 확장하는 타입을 receiver라고 부른다.
<br>위의 함수의 경우 `lastChar()`의 receiver는 `String`이다.
* 확장 함수의 body에서는 `this`를 통해 receiver에 접근할 수 있다.
* 확장 함수를 사용하기 위해서는 명시적으로 import 해야 한다.

**<u>자바 코드에서 코틀린의 확장함수 호출하기</u>**

자바에서 코틀린의 top-level 함수를 호출할 때 top-level 함수는 `static` 함수로 번역된다.
확장 함수 역시 마찬가지이다.

`under the hood it’s just a regular static function`

따라서 확장 함수는 receiver 객체의 `private` 멤버에 접근할 수 없다.

`Kotlin extension functions are regular static functions defined in a separate auxiliary class`

코틀린의 확장 함수는 자바에서 어떻게 해석될까? 아래의 확장 함수를 예로 들어보자.

```kotlin
fun String.repeat(n: Int): String {
    val sb = StringBuilder(n * length)
    for (i in 1..n) {
        sb.append(this)
    }
    return sb.toString()
}
```

위의 함수가 컴파일되어 `static` 함수가 될 때 아래와 같이 `String` 객체(the receiver)를 받을 인자가 맨 앞에 추가된다.

`public static repeat(String string, int n) { ... }`

따라서 자바에서 사용할 때는 아래와 같이 호출해야 한다(아래와 같이 번역된다).

`StringUtilKt.repeat(“ab”, 3); // ababab`

### 2. 표준 라이브러리 속 확장 함수

`Kotlin standard library = Java standard library + extensions`

Kotlin SDK는 따로 존재하지 않는다. JDK에 확장 함수를 더한 것뿐이다. 이로써 다음과 같은 장점을 얻을 수 있다.
1. small runtime jar
<br>`Kotlin doesn’t duplicate the standard implementations from Java`
2. easy Java interop

표준 라이브러리 속 확장 함수를 몇개 살펴보자.

**<u>joinToString()</u>**

```kotlin
fun <T> Iterable<T>.joinToString(
        separator: CharSequence = ", ", 
        prefix: CharSequence = "", 
        postfix: CharSequence = ""
): String
```

아래와 같이 사용한다.

```kotlin
println(listOf('a', 'b', 'c')
        .joinToString(separator = "", prefix = "(", postfix = ")")) // (abc)
```

한 번만 더 반복하자면 `joinToString()`은 확장 함수로 `String` 클래스에 정의된 함수가 아니다.

**<u>getOrNull()</u>**

```kotlin
fun <T> Array<T>.getOrNull(index: Int) = if (index in 0 until size) this[index] else null
```

**<u>withIndex()</u>**

```kotlin
val list = listOf("a", "b", "c")
for ((index, element) in list.withIndex()) {
    println("$index $element")
}
```

**<u>until()</u>**

`infix fun Int.until(to: Int): IntRange`

```kotlin
1.until(10)
1 until 10 // infix 함수이기 때문에 가능한 문법
```

`1 until 10`에서 `until`은 마치 빌트인 문법인 것 같지만
사실은 infix 형태로 호출된 확장 함수이다. infix는 나중에 살펴보도록 한다.

**<u>to()</u>**

`infix fun <A, B> A.to(that: B) = Pair(this, that)`

`mapOf(0 to “zero”, 1 to “one”)`

### 3. Calling Extensions

**<u>확장 함수를 override 할 수 있을까?</u>**

확장 함수에 대해 이해했는지 확인할 수 있는 문제가 있다.

```kotlin
open class Parent
class Child: Parent()

fun Parent.foo() = "parent"
fun Child.foo() = "child"

fun main(args: Array<String>) {
    val parent: Parent = Child()
    println(parent.foo())
}
```

parent가 출력될까 child가 출력될까? 확장 함수는 내부적으로 자바 `static` 함수이므로 다음과 같이 자바로 바꿔서 생각해보자.

```java
public static String foo(Parent parent) { return "parent"; }
public static String foo(Child child) { return "child"; }

public static void main(String[] args) {
    Parent parent = new Child();
    System.out.println(foo(parent));
}
```

확장 함수가 컴파일될 때 receiver의 타입은 자바 메소드의 첫 번째 매개변수로 변환된다.
자바는 `static` 함수를 정적으로 resolve한다. 다시 말해, 컴파일 할 때 어떤 `static` 함수를 호출하는 것인지 결정한다.
이때, 오직 인자의 타입만 보고 결정하기 때문에 답은 parent이다. 런타임에 실제 담겨 있는 객체는 아무 영향을 주지 못한다.
`static` 함수의 호출은 컴파일 타임에 이미 결정되기 때문이다.

`Extensions are static Java functions under the hood. No override for extension functions in Kotlin`

`override`할 수 있으려면 시그니처가 같아야 하는데 receiver의 타입이 다르면 매개변수의 타입이 달라지므로 시그니처 또한 달라져서 `override` 될 수가 없다.

코틀린에서도 확장 함수를 다룰 때는 `println(parent.foo())`에서 `parent`에 어떤 객체가 담겨있는지가 아니라 `parent`가 어떤 타입인지만 본다.

**<u>멤버 함수와 확장 함수가 싸우면 누가 이길까?</u>**

멤버 함수와 같은 시그니처를 갖는 extension 함수를 정의한 뒤 호출하면 어떤 함수가 호출될까?

```kotlin
fun String.get(index: Int) = '*'

fun main(args: Array<String>) {
    println("abc".get(1))
}
```

무엇이 출력될까? `*`? `b`?

`member always wins(extension is shadowed by a member)`

따라서 `b`가 출력된다.

하지만 확장 함수가 멤버 함수를 overload 하는 것은 가능하다.

### 4. 확장 함수가 중요한 이유

**<u>Extensions are often named among the most important or lovable features of Kotlin. Why are they so important?</u>**

`I think their main purpose is to keep your classes and interfaces APIs minimal.`

`It’s important to capture the essence of the abstraction with this small set of members that covers the essential functionality.`
`members for intrinsic things, extensions for everything else.`

**<u>What other use cases are extensions good for?</u>**

`you can even extend the APIs of existing Java libraries.`

어떤 라이브러리가 코틀린을 전혀 염두에 두지 않고 만들어졌다고 하더라도, extension을 추가함으로써 kotlin-like, ergonomic, idiomatic API로 바꿀 수 있다.

**<u>조언</u>**

확장 함수와 해당 확장 함수가 확장하는 클래스는 가깝게 두는게 좋다.
확장 함수가 최대한 발견하기 쉽도록 말이다. 물론 확장 함수가 일반적이지 않고 다른 모듈의 특정 서브 도메인과 관련된 확장 함수라면 해당 모듈에 둔다.
그러나 어떤 경우든 확장 함수끼리는 같은 클래스나 파일에 모아주는게 좋다.

`When you extend some library or some particular class, just put all the logically connected extensions into the same file so that they’re easily discoverable.`

### 5. TMI

주제에 안맞는 내용은 TMI라는 꼭지를 만들어서 여기에 정리해두는 게 좋을 것 같다.

**<u>Triple-quoted string literals</u>**

코틀린에서 여러 줄의 문자열을 formatting 할 수 있는 방법이다.

```kotlin
val q = """To code,
    or not to code?.."""
```

위와 같이 사용하는 경우 indent까지 들어가서 아래와 같이 출력된다.

```text
To code,
  or not to code?..
```

IDE에서 줄바꿈에 따라 추가된 indent를 무시하고 새로운 행을 시작하고 싶다면 다음과 같은 방법이 존재한다.

```kotlin
val q = """To code,
    |or not to code?..""".trimMargin() // |는 default margin prefix이며 설정할 수 있다.
```

```kotlin
val q = """To code,
    or not to code?..""".trimIndent()
```

결과는 둘 모두 다음과 같다.

```text
To code,
or not to code?..
```

**<u>toIntOrNull()</u>**

`String`의 확장 함수 중 하나이다.

```kotlin
"123".toIntOrNull() // 123
"xx".toIntOrNull() // null
"xx".toInt() // NumberFormatException
```

---
<br>처음 코틀린을 배울 때 가장 임팩트가 컸던 것을 꼽으라면 확장 함수를 꼽겠다.
당연히 클래스에 정의된 멤버 함수인 줄 알았는데, 해당 클래스와 아무 관련도 없는 사용자가
자신이 원하는 함수를 확장 함수로 만들어 해당 클래스의 멤버 함수인냥 사용하는 것이라니.
마법같은 일이라고 생각했는데 `under the hood,` 그저 `static` 함수에 불과하다니.