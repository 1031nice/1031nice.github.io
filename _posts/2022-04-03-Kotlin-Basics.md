---
layout: post
title:  Kotlin for Java Developers - Kotlin Basics
date:   2022-04-03 14:26 +0900
categories: TIL
---

### TOC
1. [Hello world example](#"Hello,-world"-example)
2. [Variables](#Variables)
3. [Functions](#Functions)
4. [Control Structures](#Control-Structures)
5. [Exceptions](#Exceptions)

### "Hello, world" example

간단한 예제를 통해 코틀린의 특징을 살펴보자.

```kotlin
fun main(args: Array<String>) {
    val name = if (args.size > 0) args[0] else "Kotlin"
    println("Hello, $name!")
}
```

- 세미콜론이 필요없다. 
- 함수는 `fun`을 이용하여 정의한다. 
- 함수가 top level(package level)에 존재할 수 있다. 
- 인자가 없는 메인 함수를 사용할 수 있다. *(since kotlin 1.3)* 
- 배열은 일반 클래스와 똑같이 생겼다. `Arrays<>`
- `if`도 표현식이다. 변수에 담길 수 있다. 표현식의 결과로 `if`의 결과값을 반환한다. 
- `String template`이라는 문법이 존재한다. 이를 사용하면 문자열에 값을 삽입할 수 있다. 
<br>복잡한 표현식을 삽입해야 하는 경우라면 `{}`를 이용하면 된다. `”...${functionalCall(...)}...”`

### Variables
`val`: value, read-only(assigned-once), 자바의 `final` 변수에 대응

`var`: variable, mutable

**<u>Local type inference</u>**

코틀린은 자바와 마찬가지로 statically-typed 언어이다. 이는 모든 변수와 모든 표현식이 하나의 타입을 갖는다는 것을 의미한다.
하지만 자바와 달리 컴파일러가 추론할 수 있는 경우 타입을 생략할 수 있다.
```kotlin
val greeting = "Hi" // val greeting: String = "Hi"
var number = 0
```


**<u>val에 담긴 객체를 수정할 수 있을까?</u>**

가능하다. `val`은 해당 변수에 어떤 값을 재할당할 수 없음을 의미한다.
`val`가 객체를 담고 있는 경우 `val`은 객체의 주소를 저장하고 있는 참조변수이기 때문에
다른 객체를 가리키도록 `val`에 어떤 다른 객체의 주소를 재할당할 수는 없으나,
`val`가 이미 가지고 있는 객체의 값을 바꾸는 것은 `val`에 담겨 있는 주소와 아무 상관이 없다.
```
val languages = mutableListOf(“Java”)
languages.add(“Kotlin”) // okay
```

**<u>모든 변수를 `val` 키워드로 선언하려고 하라</u>**

불변 레퍼런스, 불변 객체, 사이드 이펙트가 없는 함수는
코드를 더 함수형 스타일에 가깝게 만들어준다.

**<u>맥락에서 타입이 분명히 드러나지 않는다면 타입을 생략하지 마라</u>**

### Functions

- 코틀린의 함수는 아래와 같은 모습이다.
```kotlin
fun function_name(arg1: type1, arg2: type2): return_type {
    body
}
```

- 함수가 단순히 하나의 표현식을 반환하고 있다면 `function with expression body`라 불리는 문법을 사용할 수 있다.
<br>`fun max(a: Int, b: Int): Int = if (a > b) a else b`
- 리턴 타입이 생략된 함수는 `Unit`을 반환한다. 자바의 `void`에 대응된다.
- 코틀린에서는 함수를 어디서나 정의할 수 있다.

```kotlin
fun topLevel() = 1 // top-level function

class A { // member function
    fun member() = 2
}

fun other() { // local function
    fun local() = 3
}
```
- 코틀린의 top-level 함수는 바이트코드 레벨에서 보면 자바의 `static` 함수와 같다.

**<u>Named & default arguments</u>**

코틀린은 함수를 정의할 때 인자의 디폴트 값을 설정할 수 있고,
인자에 이름을 붙여 함수를 호출할 수도 있다.

```kotlin
fun displaySeparator(character: Char = '*', size: Int = 10) {
    repeat(size) {
        print(character)
    }
}
```

위와 같이 함수가 정의되어 있을 때, 아래와 같이 함수를 호출해보자.

`displaySeparator(size = 5)`

`character` 인자는 디폴트 값인 `*`로 설정되고, `size`는 호출시 명시했던 값인 5로 설정된다.


- named arguments를 줄 때는 순서가 중요하지 않지만,
named arguments가 아닌 경우 순서대로 해석한다.
- 자바의 경우 named arguments는 불가능하고,
default arguments는 메소드 오버로드를 이용해야만 가능하다.

### Control Structures

**<u>if</u>**

코틀린에서는 `if`도 표현식이다.

`val max = if (a > b) a else b`

코틀린에는 3항 연산자가 없다.

**<u>when</u>**



코틀린에는 `when`이라는 문법이 존재한다.
자바의 `switch`에 대응된다.

```kotlin
when (color) {
    BLUE -> println("cold")
    ORANGE -> println("mild")
    else -> println("hot")
}
```
* `switch`와 달리 `break`가 필요 없다.
각 브랜치의 끝에 `break`가 붙어 있다고 보면 된다.
예를 들어, `color`에 `BLUE`가 들어왔다면 "cold"를 출력하고 `when`을 빠져나간다.

* `when`의 인자를 여러 값과 비교할 수도 있다.

```kotlin
fun respondToInput(input: String) = when (input) {
    "y", "yes" -> "I'm glad you agree"
    "n", "no" -> "Sorry to hear that"
    else -> "I don't understand you"
}
```

* 타입을 검사할 수도 있다.

```kotlin
when (pet) {
    // 코틀린의 is는 자바의 instanceof에 대응된다.
    is Cat -> pet.meow() // smart casts
    is Dog -> pet.woof() // 타입 캐스팅하여 사용하지 않아도 된다
}
```

* 코틀린 1.3부터는 `when`의 인자에 변수를 선언할 수 있다.

```kotlin
when (val pet = getMyFavoritePet()) {
    is Cat -> pet.meow()
    is Dog -> pet.woof()
}
// when을 벗어나면 pet에 접근할 수 없음
```

* `when`에 어떤 인자도 주지 않을 수도 있다.
어떤 인자도 주지 않을 경우 브랜치 조건에 어떠한 불린 표현식을 넣을 수 있으며,
해당 불린 표현식이 `true`일 경우 해당 브랜치의 결과를 반환한다

**<u>Loops</u>**

코틀린 역시 `while` `do-while` `for`를 문법으로 갖는다.
`for`만 다른데, 코틀린에서는 `for`에 `in`이라는 키워드를 사용하며,
`Map`에 담긴 element를 바로 순회할 수 있다.

```kotlin
for ((key, value) in map) {
    println("$key = $value")
}
```

코틀린에서 인덱스와 함께 `for`를 사용하고 싶을 경우는 아래와 같이 사용할 수 있다.
```kotlin
val list = listOf("a", "b", "c")
for ((index, element) in list.withxIndex()) {
    println("$index: $element")
}
```

인덱스만 사용하고 싶은 경우는 아래와 같이 사용할 수 있다.
```kotlin
for (i in 1..9) {
    print(i) // 123456789
}

for (i in 1 until 9) {
    print(i) // 12345678
}

for (i in 9 downTo 1 step 2) {
    print(i) // 97531
}
```

코틀린에서는 문자열도 순회할 수 있다.
```kotlin
for (ch in "abc") {
    print(ch) // abc
}
```

**<u>in</u>**

`in`은 두 가지 사용 사례가 있다. 하나는 위에서 살펴본 것처럼 반복문 안에서 사용되는 것이고, 다른 하나는 포함 여부를 확인할 때이다.

`fun isLetter(c: Char) = c in ‘a’..‘z’ || c in ‘A’..’Z’`

`fun isNotDigit(c: Char) = c !in ‘0’..’9’`

```kotlin
fun recognize(c: Char) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter"
    else -> "I don't know…"
}
```

**<u>range</u>**

비교가 가능하다면 무엇이든 `range`를 사용할 수 있다.
```kotlin
val stringRange: ClosedRange<String> = "ab".."az" // ab, ac, ..., ay, az
val dateRange: ClosedRange<MyDate> = startDate..endDate
```

비교가능한 객체라면 `compareTo`가 아닌 비교연산자를 사용할 수 있다.

`println("java" < "kotlin") // true`

### Exceptions
코틀린에서는 어떤 예외든 발생할 수 있는 곳에서는 굳이 던질 수 있음을 표현하지 않아도 되고,
사용하는 쪽에서는 잡아서 처리하지 않아도 된다. 모든 예외가 unchecked 예외이다.

코틀린에서는 `throw`도, `try`도 표현식이다.
```kotlin
val number = try {
    Integer.parseInt(string)
} catch (e: NumberFormatException) {
    null
}
```
