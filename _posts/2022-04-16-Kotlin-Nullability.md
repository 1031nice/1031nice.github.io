---
layout: post
title:  Kotlin for Java Developers - Nullability
date:   2022-04-16 12:55 +0900
categories: TIL
---

## Nullability

### TOC
1. [Nullable types](#1-nullable-types)
2. [Nullable types under the hood](#2-nullable-types-under-the-hood)
3. [Safe casts](#3-safe-casts)
4. [Importance of nullability](#4-importance-of-nullability)
5. [TMI](#5-tmi)

### 1. Nullable types
`The problem of nullability is sometimes referred to as billion dollar mistake.` 

`NullPointerException` is really hard to fix.

`Modern approach: to make NPE compile-time error, not run-time error`

코틀린은 nullable types과 non-nullable types을 구분한다.

```kotlin
val s1: String = always not null
val s2: String = null // Kotlin compiler will complain
val s3: String? = null // 타입 뒤에 붙은 '?'가 nullable을 표현한다
```

Non-nullable types의 멤버에 접근하는거나 확장 함수를 호출하는 것은 아무 문제가 없다.
하지만 nullable type의 변수를 dereference 하려고 시도하면 컴파일 에러가 발생한다.
Nullable type은 `null`이 담겨 있을 수도 있으므로 바로 dereference를 시도하는 것은
`NullPointerException`(이하 NPE)을 일으킬 수도 있기 때문이다.
(코틀린은 nullable types과 non-nullable types를 구분함으로써 NPE를 컴파일 타임 에러로 만들었다.)

**<u>Safe access(call) expression</u>**

Nullable type의 객체를 dereference 하고 싶을 때는 어떻게 해야할까?
가장 쉬운 방법은 명시적으로 `null`이 아님을 검사한 뒤 dereference하는 것이다.

```kotlin
val s: String?

if (s != null) {
    s.length // s is now non-nullable type
}
```

하지만 safe access expression이라는 더 좋은 방법이 있다. 아래의 코드를 보자.

`s?.length // safe access(call) expression`

safe access expression `?.`를 사용하면 receiver가 `null`이 아닐 경우 access 결과를,
`null`인 경우 `null`을 표현식의 결과로 반환한다. 다시 말해, 아래와 같은 의미를 갖는다.

```kotlin
val s: String?

val length = if (s != null) s.length else null // val length = s?.length
```

**<u>Elvis separator</u>**

Safe access expression의 반환 결과를 담는 변수는 nullable 타입이어야 한다.
Safe access expression이 `null`을 반환할 수도 있기 때문이다.
`null`이 담기길 원하지 않는다면 `null`인 경우 사용할 default 값을 지정해주어야 한다.

```kotlin
val s: String?

val length: Int = if (s != null) s.length else 0
```

위와 같이 `null`이 아닌 값을 담을 수도 있지만,
Elvis separator를 이용하면 더 세련되게 위와 똑같은 일을 할 수 있다.

`val length: Int = s?.length ?: 0 // s가 null이 아닌 경우 s.length, null인 경우 0`

Elvis separator는 `?:`이며 고개를 왼쪽으로 꺾어서 바라보면 엘비스 프레슬리의 앞머리와 눈처럼 보인다고 해서 Elvis라는 이름이 붙었다. (from Groovy)

**<u>Control-flow analysis</u>**

코틀린 컴파일러는 control-flow analysis를 수행한다.
따라서 reference가 `null`인지를 명시적으로 검사하고 `fail()` 또는 `return` 같은 함수를 호출한다면
이후부터는 safe access 없이도 접근할 수 있다.

```kotlin
val s: String?

if (s == null) return

s.length // s smart cast to non-nullable type
```

**<u>Not-null assertion</u>**

Not-null assertion은 피연산자가 `null`이면 NPE를, `null`이 아니면 피연산자를 반환한다.

`s!! // not-null assertion`

Not-null assertion 이후의 코드 역시 safe access 없이도 nullable 타입의 변수에 접근할 수 있다.
하지만 not-null assertion은 개발자가 `null`이 아님을 보장하는 것이기 때문에 실수할 여지가 있다.
지금까지 nullable을 다루는 방법들은 "`null`이면 이렇게 하고, `null`이 아니면 이렇게 해주세요." 하는 태도였다면,
not-null assertion은 "`null`이 아니니까 이렇게 해."라고 볼 수 있다. 되도록이면 쓰지 않는 것이 좋다.
그럼에도 이런 문법이 존재하는 이유는 코틀린 컴파일러가 적절한 타입을 추론하지 못할 때도 있기 때문이다.

```kotlin
class MyAction {
    fun isEnabled(): Boolean = list.selectedValue != null
    // actionPeerformed()는 항상 isEnalbe() 확인 후에 호출된다
    fun actionPerformed() {
        val value = list.selectedValue!! // 따라서 list.selectedValue는 null일 수 없다
        ...
    }
}

...
if (action.isEnabled()) {
    action.actionPerformed()
}
```

Not-null assertion을 사용하면 명시적으로 어디서 NPE가 발생하는지 강조할 수 있다.
NPE가 발생하면 무엇이 원인이었는지 바로 확인할 수 있다.
때문에 둘 이상의 not-null assertion을 한 줄에 사용하는 것은 좋지 않은 선택이다.

`person.company!!.address!!.country`

위의 코드에서 NPE가 발생한다면 누가 `null`인지 NPE로부터 바로 알 수 없다. 

일반적으로 not-null assertion 보다 safe 연산자(safe access, Elvis operator, if-checks)를 사용하는 것이 낫다.

### 2. Nullable types under the hood

`under the hood,` nullable, non-nullable 타입은 `@Nullable`, `@NotNull` 어노테이션으로 구현되었다.
따라서 nullable 타입을 사용한다고 performance overhead가 생기지는 않는다.

**<u>Nullable types != Optional</u>**

- `Optional`은 Nullability 문제에 대한 다른 해결책이다.
- `Optional`은 값 또는 값의 부재를 저장하고 있는 특별한 라이브러리 클래스이다.
- `Optional`을 통해 값이 available한지 아닌지를 검사할 수 있다.
- `Optional` 타입은 초기의 객체에 대한 참조를 저장하고 있는 wrapper이다.
- `Optional` 값마다 추가적으로 객체가 생성된다.

`Optional`과 nullable 타입은 같은 문제를 해결하지만 성능 관점에서 보면 매우 다르다.
Nullable 타입은 어떤 wrapper도 생성하지 않는다. (어노테이션을 통해 구현되어 있다.)
또한 `Optional<T>`에 `T` 객체를 저장할 수 없는 반면(`Optional` 객체만 담을 수 있다),
코틀린의 nullable 타입에는 non-null 객체를 담을 수 있다.

`When you use nullable types under the hood,
the Kotlin compiler adds additional annotations which are only checked at the compilation time.
At run time, nullable string is the same string as regular Java String`

**<u>List of nullable Int vs nullable List</u>**

`List<Int?>`
List of nullable Int. 리스트는 null일 수 없다. 리스트에 담기는 원소는 nullable Int이다.

`List<Int>?`
Nullable List. 리스트 자체가 null일 수 있다. 리스트에 담기는 원소는 Int이다.

문제를 풀어보자.

`Mark the lines which require a question mark to make the code compile`

```kotlin
// 연습 문제 1
fun foo(list1: List<Int?>, list2: List<Int>?) { 
    list1.size // 1 
    list2.size // 2
    
    val i: Int = // 3 
         list1.get(0) // 4 
    val j: Int = // 5
        list2.get(0) // 6
}
```

답은 가장 마지막 꼭지에 정리해두는 게 좋겠다.

**<u>Nullable 타입과 확장 함수</u>**

Nullable 타입도 확장 함수를 가질 수 있다. 처음에는 확장 함수를 사용할 때 멤버함수처럼 사용하기 때문에
receiver가 `null`일 때 `null`의 멤버 함수를 호출하려고 하는 것처럼 보여서 NPE가 발생할 것이라고 오해했다.

하지만 지난 시간에 살펴봤듯 확장 함수란 결국 receiver 타입의 객체를 첫 번째 인자로 받는 `static` 함수와 다를 게 없다.
첫 번째 인자에 null이 들어온다고 해서(nullable 타입을 받는다고 해서) `static` 함수에 문제가 생기는 것은 아니기 때문에
nullable 타입에 확장함수를 추가하는 것은 아무 문제가 없다.

```kotlin
fun String?.isEmptyOrNull() = this == null || this.isEmpty() // smart cast

fun main() {
    val s1: String? = null
    println(s1.isEmptyOrNull()) // true
}
```

위의 예시에서 `s1`은 nullable 타입이지만 `isEmptyOrNull()`를 마치 보통의 reference처럼 호출하고 있다.
이로부터 보통의 reference처럼 멤버에 접근한다는 것이 꼭 그 참조변수가 `null`이 아니라는 것을 보장하지는 않는다는 사실을 알 수 있다.

Nullable receiver에 확장 함수를 추가할 때는 혼란을 주지 않도록 주의가 필요하다.
이름에 명시적으로 receiver가 `null`일 수 있음을 알린다거나 하는 식으로 말이다.

### 3. Safe casts

**<u>Type cast: as</u>**

```kotlin
if (any is String) {
    any.toUpperCase() // smart cast to String
}
```

**<u>Safe cast: as</u>**

`(any as? String)?.toUpperCase()`

타입 캐스트 `as`는 캐스팅에 실패할 경우 예외를 던지는 반면, safe 캐스트 `as?`는 null을 반환한다.

### 4. Importance of nullability

**<u>Why nullability is so important?</u>**

NPE는 매우 흔히 일어나는 예외이므로 잘 다루는 것이 매우 중요하다.

코틀린은 라이브러리 또는 이외의 공간이 아닌 타입 시스템에서 nullabiliity를 제어할 수 있도록 해두었다.
non-null, nullable 타입을 포함하여 nullability를 잘 다룰 수 있게 도와주는 유용한 operators와 smart cast 등은
nullability를 제어하기 쉽게 만들었을뿐 아니라 `null`을 first-class citizen으로 만들었다.

nullability를 다룰 수 없을 때 무언가의 부재를 `null`로 나타내는 것이 안티패턴이었으나 코틀린에서는 충분히 가능한 패턴이다.
코틀린의 타입 시스템에서 `null`을 추적하는 것을 도와주기 때문이다.

`Nullability in the type system is also one of the key features of Kotlin.`

### 5. TMI

연습 문제 1의 정답: 4개(2번, 3번, 5번, 6번)
