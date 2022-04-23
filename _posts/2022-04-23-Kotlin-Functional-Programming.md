---
layout: post
title:  Kotlin for Java Developers - Functional Programming
date:   2022-04-23 13:03 +0900
categories: TIL
---

## Functional Programming

### TOC
1. [Lambdas](#1.-Lambdas)
2. [Common Operations on collections](#2.-Common-Operations-on-collections)
3. [Safe casts](#3.-Safe-casts)
4. [Importance of nullability](#4.-Importance-of-nullability)

### 1. Lambdas

람다는 표현식으로 사용될 수 있는 익명 함수이다.

**<u>람다 문법</u>**

`{ parameters -> body }`

`{ x: Int, y: Int -> x + y}`

코틀린에서 람다는 항상 `{}`와 함께 사용된다.
인텔리제이에서는 편의를 위해 람다의 `{}`만 굵게 표시하여 다른 `{}`과 구분한다.

람다는 표현식으로 사용될 수 있으므로 다른 함수의 인자로 전달될 수 있다.
다른 함수의 인자로 전달할 때 아래와 같이 람다 전체를 괄호안에 둘 수 있다.

`list.any({ i: Int -> i > 0})`

하지만 람다가 마지막 인자인 경우 람다 전체를 괄호에 넣는 것보다 더 좋은 표현 방법이 있다.
아래와 같이 람다를 괄호 밖으로 위치시키는 것이다. (groovy 아이디어 차용)

`list.any() { i: Int -> i > 0 }`

람다를 밖에 두었을 때 괄호가 비었다면(함수의 인자가 람다 하나라면) 괄호를 생략할 수 있다.

`list.any { i: Int -> i > 0 }`

람다 속 인자의 타입을 추론할 수 있는 경우(맥락상 타입이 분명한 경우) 타입을 생략할 수 있다.

`list.any { i -> i > 0 }`

람다의 인자가 하나라면 `it`으로 대체할 수 있다.

`list.any { it > 0 }`

`’it’ denotes the argument if it’s only one`

**<u>Multi-line 람다</u>**

```kotlin
list.any {
    println("processing $it")
    it > 0 // last expression is the result
}
```

**<u>Destructuring declarations</u>**

람다가 쌍을 이루는 값을 인자로 받거나 `Map`의 entry를 인자로 받는 경우 destructuring 문법을 사용할 수 있다.
`for`문에서 `Map`을 순회할 때 사용하는 것과 같은 문법이다.

```kotlin
map.mapValues { entry -> "${entry.key} -> ${entry.value}!" }

map.mapValues { (key, value) -> "$key -> $value!" } // Destructuring declarations
```

`the compiler will automatically destruct entry to key and value.`

만약 람다 매개변수 중 하나가 쓰이지 않는 경우 해당 매개변수의 이름을 `_`로 대체할 수 있다. 가독성도 좋고, 사용하지 않을 값을 위해 이름을 짓지 않아도 돼서 편리하다.

`map.mapValues { (_, value) -> “$value!” }`

### 2. Common Operations on collections

설명보다는 예제 위주로 보는 게 좋을 것 같다.

```kotlin
data class Hero (
    val name: String,
    val age: Int,
    val gender: Gender?
)
enum class Gender { MALE, FEMALE }

val heroes = listOf(
  Hero("The Captain", 60, MALE),
  Hero("Frenchy", 42, MALE), 
  Hero("The Kid", 9, null), 
  Hero("Lady Lauren", 29, FEMALE), 
  Hero("First Mate", 29, MALE), 
  Hero("Sir Stephen", 37, MALE))
```

**<u>firstOrNull</u>**

`heroes.firstOrNull { it.age == 30 }?.name // null`

직관적이다.

**<u>map</u>**

`heroes.map { it.age }.distinct().size // 5`

`map`은 기존 컬렉션 원소를 변환하여 새로운 컬렉션을 만든다. 위 예제의 경우 `map`의 결과 `Hero` 객체가 담긴 컬렉션에서 `Hero` 객체의 나이를 원소로 하는 새로운 컬렉션이 얻어진다. `map`은 기존의 컬렉션 원소를 변환하는 것이므로 컬렉션의 크기가 변하지는 않는다.

**<u>distinct</u>**

중복된 원소를 제거한 결과를 반환한다. 역시 직관적이다.

**<u>filter</u>**

`heroes.filter { it.age < 30 }.size`

`filter`는 `Predicate`를 인자로 받아 참인 원소만 남긴 결과를 반환한다. 위 예제의 경우 `filter` 결과 나이가 30 미만인 히어로만(컬렉션) 남게된다.

**<u>partition</u>**

`val (youngest, oldest) = heroes.partition { it.age < 30 }`

참인 원소만 남겼던 `filter`와 달리 `partition`은 `Predicate`를 인자로 받아 참인 원소와 거짓인 원소를 분리하여 두 개의 컬렉션을 반환한다. 이때 순서는 참인 컬렉션, 거짓인 컬렉션 순이다.

**<u>maxBy, minBy</u>**

`heroes.maxBy { it.age }?.name // The Captain`

원소 자체가 비교가능하지는 않으나 원소의 특정 프로퍼티 또는 다른 지정된 방법에 의해 비교 가능할 때 `maxBy`와 `minBy`를 사용할 수 있다.

**<u>all, none, any</u>**

`heroes.all { it.age < 50 } // false`

`heroes.any { it.gender == FEMALE } // true`

`all`, `none`, `any`는 `Predicate`을 인자로 받아 참 또는 거짓을 반환한다. 순서대로 보면 주어진 `Predicate`를 모든 원소가 만족하는지, 모든 원소가 만족하지 않는지, 적어도 하나의 원소는 만족하는지에 대해 참 또는 거짓을 반환한다.

이 predicates는 interchangeable하다. 상황에 맞는 가장 단순한 predicate를 사용하는 것이 좋다.
반대되는 조건을 원한다고 해서 꼭 표현식 전체를 부정(negate)할 필요는 없다. 다음의 예를 보자.

```kotlin
fun List<Int>.allNonZero() = all { it != 0 } // 1
fun List<Int>.allNonZero1() = none { it == 0 } // 2
fun List<Int>.allNonZero2() = !any { it == 0 } // 3

fun List<Int>.containsZero() = any { it == 0 } // 4
fun List<Int>.containsZero1() = !all { it != 0 } // 5
fun List<Int>.containsZero2() = !none { it == 0 } // 6
```

`allNonZero()`와 `containsZero()`는 서로 원하는 조건이 정확히 반대이다.
따라서 1번을 negate하여 5번과 같이 predicate를 이용할 수도 있지만 4번과 같이 negate 없이도 동일한 조건을 검사할 수 있다.
이럴 때는 4번과 같이 negate 없는, 가장 단순한 predicate를 사용하는 것이 좋다. negation은 읽거나 이해하는데 노력이 필요하기 때문이다.

**<u>groupBy</u>**

```kotlin
val mapByAge: Map<Int, List<Hero>> = 
  heroes.groupBy { it.age }
val (age, group) = mapByAge.maxBy { (_, group) -> 
  group.size
}!!
println(age) // 29
```

`heroes.groupBy { it.age }`의 결과 아래와 같이 히어로의 나이를 key, 해당 나이인 히어로(컬렉션)를 value로 하는 `Map`이 만들어진다.

```
9 -> Hero(The Kid)
29 -> Hero(Lady Lauren), Hero(First Mate)
…
```

그리고 `mapByAge.maxBy { (_, group) -> group.size }!!`의 결과 히어로의 수가 가장 많은 나이(key)를 갖는 엔트리가 반환된다. 
따라서 `println(age)`의 결과는 29이다.

**<u>associateBy</u>**

```kotlin
val mapByName: Map<String, Hero> = 
  heroes.associateBy { it.name }
mapByName["Frenchy"]?.age // 42
```

`associateBy`는 `groupBy`와 유사하나 둘 이상의 value가 같은 key를 갖는 경우 마지막 value만 key에 매핑되어 저장된다.
전자는 `key -> value(컬렉션)`을 entry로 갖는 `Map`이었다면 후자는 `key -> value(유일)`을 entry로 갖는 `Map`이다.
이 예제에서 히어로의 이름은 겹치지 않으므로 히어로가 덮어써지는 일은 없다.

```
“The Kid” -> Hero(The Kid)
“Lady Lauren” -> Hero(Lady Lauren)
“First Mate” -> Hero(First Mate)
…
```

따라서 `mapByName["Frenchy"]?.age`의 결과는 42이다. 참고로 `Map`의 원소에 접근하는 방법은 아래와 같이 두 가지가 있다.

```kotlin
mapByName["unknown"]?.age // null
mapByName.getValue("unknown").age // NoSuchElementException
```

**<u>associate</u>**

```kotlin
val mapByName = heroes.associate { it.name to it.age }
mapByName.getOrElse("unknown") { 0 }
```

만들어지는 `Map`의 value가 항상 컬렉션의 원소였던 `associateBy`와 달리 `associate`는 value 또한 지정할 수 있다.
즉, `associateBy`의 value는 `Hero` 객체만 가능하지만 `associate`의 value는 원하는 대로 지정할 수 있다. 
`val mapByName = heroes.associate { it.name to it.age }`의 결과 아래와 같은 컬렉션이 만들어진다.

```
“The Kid” -> 9
“Lady Lauren” -> 29
“First Mate” -> 29
…
```

**<u>getOrElse</u>**

`getOrElse`는 key와 람다를 인자로 받아 key가 존재할 경우 대응되는 value를 반환하고, 존재하지 않을 경우 람다의 결과를 반환한다.
`mapByName.getOrElse(“unknown”) { 0 }`는 0을 반환한다.

**<u>flatMap</u>**

```kotlin
// an example of bad code
val (first, second) = heroes
  .flatMap { heroes.map { hero -> it to hero } } 
  .maxBy { it.first.age - it.second.age }!!
first.name
```

`flatMap`은 먼저 `map`처럼 원래의 컬렉션에 주어진 변환 함수를 적용하여 새로운 컬렉션을 만들고, 
새로운 컬렉션의 모든 원소를 하나의 리스트로 반환한다. 다음은 `flatMap`에 대한 설명이다.

`Returns a single list of all elements yielded from results of transform function being invoked on each element of original array.`

위의 예제 코드 중에서 `heroes.flatMap { heroes.map { hero -> it to hero } }`만 떼어놓고 보자.
`it`은 `flatMap` 람다의 인자이고, `hero`는 안쪽 `map` 함수의 람다의 인자이다.
따라서 둘 모두 `it`을 그대로 쓰는 대신 다음과 같이 이름을 붙여주는 게 더 읽기 좋다.

`heroes.flatMap { first -> heroes.map { second -> first to second } }`

결국 모든 히어로 쌍의 리스트를 얻는 과정이었다. 예를 들어, 히어로 A, B, C가 있다면 위의 결과 (A, A), (A, B), (A, C), (B, B), (B, C), … 가 얻어진다.

`maxBy`는 `flatMap` 뒤에 등장하므로 결국 모든 히어로 쌍에 대해 람다의 결과가 최대인 원소를 찾겠다는 뜻이었다.
가독성을 위해 아래와 같이 분리해서 보자.

```kotlin
val allPossiblePairs = heroes
  .flatMap { first -> heroes.map { second -> first to second } } 

val (first, second) = allPossiblePairs.maxBy { it.first.age - it.second.age }!!
first.name
```

`maxBy`는 해독하기 쉽다. 모든 쌍 중에서 첫 번째 히어로와 두 번째 히어로의 나이차가 가장 큰 쌍을 반환한다.

궁극적으로 원하는 것은 결국 가장 나이 많은 히어로의 이름이었다.
이 지난한 과정을 아래와 같이 이미 존재하는 라이브러리를 이용하여 완전히 동일한 결과를 매우 쉽게 얻을 수 있다.

```kotlin
val oldest = heroes.maxBy { it.age }
oldest.name
```

**<u>가독성을 높이기 위한 팁</u>**

`Don’t use it if it has different types in neighboring lines. The it parameter name is good if the Lambda is trivial and straightforward.`

`Prefer explicit parameter name if it might be confusing otherwise`

`Learn the library and try to reuse the library functions as much as possible. Not try to re-implement all the methods.`

### Function Types

코틀린에서는 람다를 변수에 담을 수 있다.

`val sum = { x: Int, y: Int -> x + y }`

이때 변수의 타입은 무엇일까? 변수의 타입을 명시적으로 쓸 경우 아래와 같다.

`val sum: (Int, Int) -> Int = { x, y -> x + y }`

`(Int, Int) -> Int`가 바로 위 함수의 타입이다.

주의해야 할 점은 람다를 변수에 담는 것과, 람다의 결과를 변수에 담는 것은 다르다는 점이다.
람다를 변수에 담았을 때는 해당 변수(function type의 변수)를 마치 함수인 것처럼 호출할 수 있다.

```kotlin
val isEven: (Int) -> Boolean = { i: Int -> i  % 2 == 0 } // 람다를 저장

val result: Boolean = isEven(42) // true
```

람다 호출을 미루고 싶은 경우 위와 같이 변수에 람다를 저장해두고 나중에 호출할 수 있다.
(람다의 결과를 변수에 담을 경우 람다가 호출되어야 해당 변수의 값을 초기화할 수 있으므로 람다 호출을 나중으로 미룰 수 없다.)

또한 람다를 변수에 담은 경우 함수 타입의 표현식을 인자로 받는 함수에 인자로 넘길 수도 있다. `any`, `filter`와 같이 함수형 스타일로 컬렉션에 사용될 수 있는 모든 함수가 해당된다.

**<u>람다 바로 호출하기</u>**

`{ println(“hey!”) }()`

람다 중괄호 뒤에 괄호를 붙여 바로 람다를 호출할 수는 있으나 보기 어색하다. 람다를 바로 사용하고 싶은 경우 아래와 같이 `run`을 이용하는 게 더 가독성이 좋다.

`run { println(“hey!”) }`

```kotlin
val isEven = { i: Int -> i  % 2 == 0 }

val list = listOf(1, 2, 3, 4)
list.any(isEven) // true
list.filter(isEven) // [2, 4]
```

**<u>Function types and nullability</u>**

`() -> Int? vs (() -> Int)?`

전자의 경우 `null`이 담길 수는 없으나(함수 자체가 `null`인 경우) 리턴 타입이 `null`인 함수는 담을 수 있다. 후자의 경우 `null`이 담길 수는 있으나(함수 자체가 `null`인 경우) 리턴 타입이 `null`인 함수는 담을 수 없다.

**<u>Working with a nullable function type</u>**

nullable 함수 타입의 변수는 어떻게 호출할까?

```kotlin
val f: (() -> Int)? = null

// f()
if (f != null) {
    f()
}

f?.invoke()
```

### Member Reference
Member reference는 단순히 멤버 함수를 호출하거나 멤버 프로퍼티를 반환하는 람다를 대체할 수 있다.

```kotlin
people.maxBy { it. age }
people.maxBy(Person::age)
```

코틀린에서는 람다를 변수에 저장할 수는 있지만, 함수를 변수에 담을 수는 없다.
하지만 함수의 레퍼런스를 변수에 저장할 수는 있다. 어떤 함수든 해당 함수의 레퍼런스를 변수에 담아 두었다가 나중에 호출할 수 있다.

```kotlin
fun isEven(i: Int): Boolean = i % 2 == 0

val predicate = ::isEven // 함수 레퍼런스 저장하기

val predicate = { i: Int -> isEven(i) } // 이와 같이 람다에서 원하는 함수를 호출하고 람다를 저장하는 방법도 있다
```

참조하는 멤버가 프로퍼티이거나 하나 이하의 인자를 받는 함수인 경우 member reference 문법이 람다 문법에 비해 그리 간결해보이지는 않는다.
하지만 참조하는 멤버가 여러 인자를 받는 함수인 경우 차이가 크다.

```kotlin
val action = { person: Person, message: String -> sendMail(person, message) }
val action = ::sendMail // 컴파일러가 타입을 추론할 수 있으므로 매개변수를 명시하지 않아도 됨
```

**<u>Bound & non-bound references</u>**

Non-bound reference
- 인스턴스에 관계 없이 사용하는 member reference
- 해당 타입의 인스턴스이기만 하면 호출될 수 있다
- 호출마다 인스턴스까지 넘겨줘야 한다

```kotlin
val agePredicat: (Person, Int) -> Boolean =
    Person::isOlder // { person, ageLimit -> person.isOlder(ageLimit) }

val alice = Person("Alice", 29)
agePredicate(alice, 21) // non-bound이므로 isOlder를 호출할 객체까지 넘겨주어야 함
```

Bound reference
- 특정 인스턴스에 붙은 member reference
- 특정 인스턴스를 저장하고 있으므로 인스턴스를 전달하지 않는다

```kotlin
val alice = Person("Alice", 29)
val agePredicate: (Int) -> Boolean =
    alice::isOlder // { ageLimit -> alice.isOlder(ageLimit) }
agePredicate(21)
```

Bound reference는 `this` 레퍼런스에 바운드될 수도 있다.
```kotlin
class Person(val name: String, val age: Int) {
    fun isOlder(ageLimit: Int) = ...
    fun getAgePredicate() = this::isOlder // this는 생략될 수 있다
}
```

참고로 left-hand가 없는 function reference라고 해서 모두 bound reference인 것은 아니다. top-level 함수에 대한 function reference도 left-hand가 없다.

### return from Lambda

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> { // 1
    return list.flatMap { // 2
        if (it == 0) return listOf() // 3
        listOf(it, it)
    }
}
```

위와 같이 함수안에서 람다를 사용하는 경우 `return`을 사용할 때 조심해야 한다. 3번 라인의 `return`을 람다를 끝내는 `return`으로 생각할 수 있지만 실은 함수를 끝내는 `return`으로 해석되기 때문이다.

왜 이런식으로 동작하는걸까?

```kotlin
fun containsZero(list: List<Int>): Boolean {
    for (i in list) {
        if (i == 0) return true
    }
    return false
}
```

위 함수에서 for를 foreach로 바꿔보자.

```kotlin
fun containsZero(list: List<Int>): Boolean {
    list.forEach {
        if (i == 0) return true // 함수를 끝내는게 자연스럽다
    }
    return false
}
```

코틀린에서 왜 람다 속 return이 바깥 함수에 걸리게 했는지 이해했다. 그럼 어떻게 해야 람다 속 return이 람다에 걸리도록 할 수 있을까? label return 문법을 사용하면 된다.

```kotlin
list.flatMap {
    if (it == 0) return@flatMap listOf<Int>() // return@람다를_호출한_함수이름(디폴트)
    listOf(it, it)
}

list.flatMap l@{
    if (it == 0) return@l listOf<Int>() // return@정한_이름
    listOf(it, it)
}
```

다른 방법도 있다. 지역 함수를 사용하는 것이다.

```kotlin
fun duplicateNonZeroLocalFunction(list: List<Int>): List<Int> {
    fun duplicateNonZeroElement(e: Int): List<Int> {
        if (i == 0) return listOf()
        return listOf(e, e)
    }
    return list.flatMap(::duplicateNonZeroElement)
}
```

또는 익명 함수를 사용하는 것도 방법이다.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap(fun(e): List<Int> {
        if (i == 0) return listOf()
        return listOf(e, e)
    })
}
```

참고로 바이트코드 레벨에서는 람다와 익명 함수가 같다.

`return`을 사용하지 않을 수 있다면 사용하지 않는 것도 방법이다.

```kotlin
fun duplicateNonZero(list: List<Int>): List<Int> {
    return list.flatMap(fun(e): List<Int> {
        if (i == 0)
                listOf()
        else
                listOf(e, e)
    })
}
```

람다에서의 리턴(라벨을 이용하여 람다에 걸리는 리턴)은 반복문에서의 `continue`와 같은 맥락이다. 현재의 반복만 건너뛰는 `continue`처럼 람다에서는 해당 원소만 건너뛴다.

### Is Kotlin a functional language?

코틀린은 `Haskell`과 같은 순수 함수형 언어는 아니다.
현대 주요 언어는 여러 패러다임의 아이디어를 섞는 경향이 있다.
코틀린도 그렇다. OOP 아이디어, structural 아이디어, 함수형 프로그래밍 아이디어...
따라서 클래스나 인터페이스를 사용할 때는 OOP면에 있는 것이고,
function type, 람다 또는 다른 코틀린의 함수형 특징을 사용할 때는 함수형 프로그래밍면에 있는 것이다.

`I would say it's more of a style issue, so it's not about being a functional language but it's about enabling the functional style of programming.
For example, Kotlin promotes immutable data, so you are encouraged to use immutable things.
But immutable things are also first-class citizens in Kotlin, you can use them freely whenever you like, it's a matter of style.
So, I would say if you rely on immutability, if you use higher-order functions, and lambdas, and function types of course, you're doing Kotlin in the functional style.
And it's very often beneficial, so I totally recommend it.`

