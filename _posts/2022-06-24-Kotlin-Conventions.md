---
layout: post
title:  Kotlin for Java Developers - Conventions
date:   2022-06-24 16:30 +0900
categories: TIL
---

## Conventions

### TOC
1. [Operator Overloading](#1-operator-overloading)
2. [Conventions](#2-conventions)
3. [(Not)using operator overloading](#3-notusing-operator-overloading)

### 1. Operator Overloading
커스텀한 타입에 대해 산술연산자(`+`, `-`, `*,` `/`, `%`)를 오버로드하여 사용할 수 있다.

`a + b`는 `a.plus(b)`와 같다. 좌표평면 위의 점을 표현하는 `Point`라는 클래스가 있다고 할 때,
`Point`에 대해 산술연산자를 오버로드하여 사용하는 예시를 살펴보자.

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}

// ...

Point(1, 2) + Point(2, 3)
```

단항 연산자 및 대입 연산자도 오버로드할 수 있다.

`-a`는 `a.unaryMinus()`와 같다.

```kotlin
operator fun Point.unaryMinus() = Point(-x, -y)

// ...

-Point(3, 4)
```

### 2. Conventions
**<u>Comparisons</u>**

문자열을 포함하여 comparative 메소드를 정의한 클래스는 모두 아래와 같이 비교할 수 있다.

`”abc” < “def”`

Under the hood, 비교 연산자는 comparative 메소드 비교로 컴파일된다.

`a > b`는 `a.compareTo(b) > 0`와 같다.

`Comparable`을 구현할 경우 모든 곳에서 비교연산자를 사용할 수 있고,
특정 접근지시자와 함께 extension 함수로 `compareTo`를 구현하는 경우
`compareTo`가 보이는 곳에서만 비교연산자를 사용할 수 있다.

**<u>Equality check</u>**

`Under the hood, equality check calls equals()`

```kotlin
s1 == s2 // s1.equals(s2)
null == "abc" // false
null == null // true
```

코틀린의 equality check은 nullable 값에 대한 처리도 해주기 때문에
동등성 비교 전에 변수가 `null`이 아닌지 검사하지 않아도 된다.

```kotlin
// kotlin
val str : String? = null
str.equals(null) // true

// java
String str = null;
str.equals(null); // NPE
```

**<u>Accessing elements by index: []</u>**

`x[a, b]`는 `x.get(a, b)`와 같다.
`x[a, b] = c`는 `x.set(a, b, c)`와 같다.

get, set operator 함수를 커스텀한 클래스의 멤버 또는 extension 함수로 정의할 수 있다.
정해진 크기의 좌표평면을 나타내는 클래스 `Board`가 있다고 할 때 get, set을 정의하여
`[]`를 가지고 원소에 접근하는 예시를 살펴보자.

```kotlin
class Board { ... }

board[1, 2] = 'x'
board[1, 2] // 'x'

operator fun Board.get(x: Int, y: Int): Char { ... }
operator fun Board.set(x: Int, y: Int, value: Char) { ... }
```

커스텀 클래스를 2차원 배열 다루듯 `[]`를 이용해 원소에 접근할 수 있다.

**<u>The `in` convention</u>**

`a in c`는 `c.contains(a)`와 같다.

**<u>The `rangeTo` convention</u>**

`start..end`는 `start.rangeTo(end)`와 같다.

**<u>The `iterator` convention</u>**

코틀린의 문자열은 아래와 같이 `iterator` 연산자가 extension 함수로 정의되어 있다.

`operator fun CharSequence.iterator(): CharIterator`

때문에 for문을 돌 수 있다.

```kotlin
var str = "kotlin for java developers"
for (ch in str) {
    println(ch)
}
```

**<u>Destructing declaration</u>**

`val (a, b) = p`는 아래와 같다.

```
val a = p.component1() // component1()은 수도코드가 아니라 실제 함수 이름
val b = p.component2()
```

다시 말해, `componentN()` 함수가 정의되어 있다면 `val (a, b) = p`과 같은 문법을 사용할 수 있다.

변수를 정의하는 것과 구분하기 위해 destructuring을 정의할 때는 항상 괄호와 함께 사용한다.

```
Two parameters: { a, b -> … }
A destructured pair: { (a, b) -> … }
A destructured pair and another parameter: { (a, b), c -> … }
```

data 클래스는 `componentN()` 함수도 기본으로 생성해준다. 따라서 data 클래스는 destructuring 선언의 right hand에 올 수 있다.

```kotlin
data class Contact(
    val name: String,
    val email: String,
    val phoneNumber: String
) {
    // generated methods
    fun component1() = name
    fun component2() = email
    fun component3() = phoneNumber
}

// destructuring declaration의 right hand에 data 클래스 Contact가 놓임
// 사용하지 않는 변수의 경우 _를 이용하여 필요없음을 문법으로 명시 가능
val (name, _, phoneNumber) = contact
```

### 3. (Not)using operator overloading

`when to use and when not to use operator overloading?`

`my advice would be to only use operator overloading on the abstractions that have natural, commonly acknowledged operations signified by the symbols that we have in the language. So, use it with care and it will provide for readable intuitive code.`