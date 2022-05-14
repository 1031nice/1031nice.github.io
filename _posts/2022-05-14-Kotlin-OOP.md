---
layout: post
title:  Kotlin for Java Developers - OOP
date:   2022-05-14 15:29 +0900
categories: TIL
---

## Object-oriented Programming

### TOC
1. [OOP in Kotlin](#1-oop-in-kotlin)
2. [Constructors, Inheritance syntax](#2-constructors-inheritance-syntax)
3. [Class modifiers](#3-class-modifiers)
4. [Objects, object expression & companion objects](#4-objects-object-expression--companion-objects)
5. [Constants](#5-constants)
6. [Generics](#6-generics)
7. [OOP design choices](#7-oop-design-choices)
8. [TMI](#8-tmi)

### 1. OOP in Kotlin

새로운 개념은 없다. 문법이 좀 다를 뿐이다.

**<u>The defaults are different</u>**

코틀린의 디폴트는 다음과 같다.

`Any declaration is public and final by default`

* non-final로 만들고 싶다면 명시적으로 `open`이라는 키워드를 붙여야 한다.
* `public`이 아닌 다른 접근제어자를 사용하고 싶은 경우에만 명시적으로 선언한다.
  * `public`: `public` 클래스 멤버 또는 `public` 탑 레벨 declaration은 모든 곳에서 접근할 수 있다.
  * `internal`: `internal` 클래스 멤버 또는 `internal` 탑 레벨 declaration은 같은 모듈에서만 접근할 수 있다.
  * `protected`: `protected` 클래스의 멤버는 해당 클래스 내에서 또는 하위 클래스 내에서 접근할 수 있다. `protected`는 탑 레벨 declaration에 사용할 수 없다. (자바의 `protected`는 같은 패키지에서도 접근할 수 있다.)
  * `private`: `private` 클래스의 멤버는 해당 클래스 내에서만 접근할 수 있고, `private` 탑 레벨 declaration은 해당 파일에서만 접근할 수 있다.`

*코틀린에는 자바의 package-private visibiility가 없다. 대신 `internal`이라는 키워드를 통해 모듈 안에서 접근가능함을 보일 수 있다.*

*모듈이란 함께 컴파일되는 코틀린 파일의 집합이다.
인텔리제이 IDEA 모듈일 수도, 메이븐 프로젝트일 수도, 그래들 소스 집합일 수도 있다.*

*`internal`은 JVM에서 `public`과 name mangling을 통해 표현된다.*

```kotlin
class MyClass {
    internal fun foo() {}
}

// Under the hood:
public final class MyClass {
    // 실수로라도 호출되는 일이 없도록 매우 긴이름을 갖는다
    // name mangling
    public final void foo$production_sources_for_module_examples_main()
}
```

**<u>Packages</u>**

하나의 파일(.kt)에 여러 클래스를 둘 수 있다.

`One file may contain several classes and top-level functions`

코틀린에서는 패키지 이름과 디렉토리 구조가 일치하지 않아도 된다.
패키지 이름은 무엇이든 상관없으나 찾아가기 쉽도록 (디렉토리 구조와 완벽히 일치시키진 않더라도) 디렉토리 구조에 맞추는 게 좋다.

```
// java
org
-company
--store
---City.java
---Customer.java
…

// kotlin
store
-StoreModel.kt // package org.company.store
```

### 2. Constructors, Inheritance syntax

**<u>Default constructor</u>**

`A constructor is a special function used to initialize a newly created object.`

코틀린에서는 `new` 연산자를 사용하지 않고, 일반 함수를 호출하듯 생성자를 호출한다.
일반 함수는 소문자로 시작하고, 생성자는 대문자로 시작하기 때문에 구분할 수 있다.
클래스에 생성자를 정의하지 않을 경우, 코틀린 컴파일러가 인자가 없는 디폴트 생성자를 만들어준다.

**<u>Concise primary constructor</u>**

`class Person(val name: String, val age: Int)`

Primary 생성자는 위와 같이 정의할 수 있다. 위 코드 한줄을 통해 `name`, `age` 두개의 프로퍼티를 가지며,
문자열과 정수 인자를 받아 `name`과 `age` 값으로 설정하는 생성자를 갖는 클래스를 정의할 수 있다.

**<u>Full primary constructor syntax</u>**

```
class 클래스_이름(생성자_매개변수_이름: 생성자_매개변수_타입) {
    init {
        // constructor body
    }
}
```

```kotlin
class Person(name: String) {
    val name: String
    init {
        this.name = name
    }
}
```

생성자 매개변수의 앞에 `val` 또는 `var`를 붙이면 자동으로 프로퍼티를 생성한다.
따라서 위의 클래스 정의는 아래와 같이 간결하게 표현할 수 있다.

`class Person(val name: String)`

**<u>Changing visibility of a constructor</u>**

생성자의 visibility를 변경하기 위해선 `constructor` 키워드를 명시적으로 사용해야 한다.

`class InternalComponent internal constructor(name: String) { … }`

**<u>Secondary constructor</u>**

Secondary 생성자는 primary 생성자 또는 다른 secondary 생성자를 반드시 호출해야 한다.
다른 생성자는 아래와 같이 `this` 키워드를 통해 호출할 수 있다.

```kotlin
class Rectangle(val height: Int, val width: Int) { // primary
    constructor(side: Int) : this(side, side) { ... } // secondary
}
```

*Primary 또는 secondary 생성자가 존재하는 경우 디폴트 생성자는 생성되지 않는다.*

**<u>Different syntax for inheritance</u>**

어떤 클래스가 다른 클래스를 상속받거나 특정 인터페이스를 구현하는 클래스임을 표현할 때 자바에서는
각각 `extends`, `implements`라는 키워드를 사용했으나 코틀린에서는 모두 콜론(`:`)을 사용한다.

`Use colon to replace both extends and implements keywords`

```kotlin
interface Base
class BaseImpl : Base

open class Parent1
class Child : Parent1() // constructor call

open class Parent2(val name: String)
class Child2(name: String) : Parent(name)

class Child3 : Parent {
    // secondary 생성자에서 부모 생성자를 호출할 때는 super 키워드를 사용한다
    constructor(name: String, param: Int) : super(name)
}
```

**<u>Initialization order</u>**

자식 클래스의 객체를 생성하면 부모 클래스의 생성자가 호출되고 부모 클래스의 property가 초기화된 뒤 자식 클래스의 생성자가 호출되고 자식 클래스의 property가 초기화된다.

자식 클래스는 부모의 모든 property(접근할 수 없는 property도 포함)를 상속받는다. 자식 클래스의 객체를 제대로 생성하기 위해서 컴파일러는 부모 클래스로부터 상속받은 모든 property를 초기화해야 하기 때문에 부모 클래스의 생성자를 먼저 호출한다.

**<u>Overriding a property</u>**

자바와 마찬가지로 코틀린에서도 필드를 override 할 수는 없다. 따라서 property를 override할 때 필드가 아닌 getter를 override한다.

부모 클래스의 property와 해당 property를 override 하려는 자식 클래스의 property가 모두 필드를 갖는 경우 예상치 못한 결과가 나올 수 있다.

```kotlin
open class Parent {
    open val foo = 1
    init {
        println(foo)
    }
}

class Child: Parent() {
    override val foo = 2
}

fun main() {
    Child() // 0
}
```

하나씩 살펴보자. 먼저 `Parent` 클래스의 `foo` getter는 trivial하므로 field가 생성되고, `open`이므로 override 될 수 있기 때문에 `println(foo)`는 getter 호출로 해석된다.

*[Backing fields 참고](../../../2022/04/30/Kotlin-Properties.html#4-tmi)*
```java
public class Parent {
    private final int foo = 1;
    public int getFoo() { return foo; }
    public Parent() {
        System.out.println(getFoo()); // getter로 호출로 해석
    }
}
```

이제 다시 생각해보자. 자식 클래스 객체를 생성하면 부모 생성자가 먼저 호출된다. 부모 생성자에서는 `foo` property의 getter를 호출한다. `foo` property의 getter는 자식 클래스에서 재정의되어 있으므로 자식 클래스의 getter가 호출된다. 자식 클래스의 getter는 자식 클래스에 `foo` 필드가 있으므로 해당 필드의 값을 출력하려고 한다. 하지만 현재는 부모 생성자가 실행되는 중이므로 아직 자식 property는 초기화되기 전이다. 따라서 0이 호출된다.

이펙티브 자바 3판에서 부모 생성자에서 재정의 될 수 있는 멤버(non-final)를 사용하지 말라고 했던 것 같은데 같은 맥락인 것 같다.

`NPE is possible in pure Kotlin.`

위의 예시에서는 primitive 타입의 필드에 대해 단순히 getter만 호출했기 때문에 NPE는 발생하지 않고 초깃값이 출력되었지만
객체를 담고 있는 필드에 대해 getter를 통해 객체를 가져오고, 해당 객체의 멤버를 사용하려고 하면 NPE를 발생시킬 수 있다.

### 3. Class modifiers

**<u>enum class</u>**

정해진 개수의 객체를 갖는 클래스가 필요한 경우 이늄을 이용할 수 있다.

자바와 달리 이늄은 클래스와 구분되지 않고 class에 `enum` modifier를 붙여 나타낸다.

**<u>enum class with properties</u>**

이늄 클래스에 멤버 함수 및 프로퍼티를 정의할 수 있다.

```kotlin
enum class Color (
    var r: Int, var g: Int, var b: Int
) {
    // 이늄 상수 리스트와 멤버 리스트를 구분하기 위해 세미콜론 필수
    // Color라는 클래스의 객체는 오직 아래 3개만 존재 
    BLUE(0, 0, 255), ORANGE(255, 165, 0), RED(255, 0, 0);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

**<u>data class</u>**

`data` modifier를 붙이면 `equals`, `hashCode`, `copy`, `toString`을 포함하여 몇몇의 메소드가 생성된다. `equals`, `hashCode`, `toString`과 같은 메소드를 생성할 때 컴파일러는 primary 생성자에 정의된 프로퍼티만 사용한다.

**<u>sealed class</u>**

클래스의 계층 구조를 제한한다.
`sealed` modifier가 붙으면 모든 하위 클래스가 같은 파일에 존재함을 뜻한다.

사용 예시를 살펴보자.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

// compile error: 'when' expression must be exhaustive, add necessary 'else' branch
fun eval(e: Expr): Int = when(e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
}
```

`Expr`의 하위 클래스가 `Num`과 `Sum` 뿐이라는 보장이 없다. 다른 곳에서 `Expr`을 구현한 클래스가 있을 수 있기 때문이다. 따라서 컴파일 에러를 해결하기 위해선 아래와 같이 else를 추가해야 한다.

```kotlin
fun eval(e: Expr): Int = when(e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
    else -> throw IllegalArgumentException("Unknown expression")
}
```

`Expr`에 `sealed` modifier를 붙이면 else 브랜치가 없어도 에러가 발생하지 않는다.
`sealed` modifier가 `Expr`의 모든 하위 클래스는 `Expr` 클래스가 위치한 파일에 존재함을 보장하므로
`Num`과 `Sum`이외의 `Expr` 구현체가 없음을 보장할 수 있기 때문이다.

**<u>Inner and nested class</u>**

- 자바의 Nested class: `static class A`
- 자바의 Inner class: `class A (by default)`
- 코틀린의 Nested class: `class A(by default)`
- 코틀린의 Inner class: `inner class A`

Inner class는 outer class에 대한 레퍼런스를 저장하고 있다.

자바에서 `static` 붙이는 것을 까먹는다면 inner 클래스가 되고, 이는 outer 클래스의 객체에 대한 레퍼런스를 갖게 된다. Outer 클래스 객체를 더이상 쓰지 않음에도 inner 클래스에서 레퍼런스를 저장하고 있어서 Outer 클래스 객체가 정리되지 않는 메모리 leak으로 이어질 수 있다.

코틀린에서는 nested class가 디폴트이다. 명시적으로 outer 클래스 객체에 대한 레퍼런스를 갖는 inner class로 만들고 싶다면 `inner` modifier를 붙여야 한다.

Inner class에서 outer 클래스의 `this` 레퍼런스에 접근할 때는 아래와 같이 `this` 레퍼런스에 라벨을 붙이면 된다.

```kotlin
class A {
    class B
    inner class C {
        ...this@A...
    }
}
```

**<u>Class delegation</u>**

위임하는 일이 전부인 클래스를 구현한다고 생각해보자.

```kotlin
interface Repository {
    fun getById(id: Int): Customer
    fun getAll(): List<Customer>
}

interface Logger {
    fun logAll()
}

class Controller(
    val repository: Repository,
    val logger: Logger
) : Repository, Logger {
    // 모든 메소드를 구현하는 게 아니라 다른 인스턴스에 위임하는 일이 전부
    override fun getById(id: Int): Customer = repository.getById(id)
    override fun getAll(): List<Customer> = repository.getAll()
    override fun logAll() = logger.logAll()
}

```

위와 같이 일일이 위임하는 코드를 작성할 수도 있지만 class delegation을 통해 아래와 같이 간편하게 같은 일을 할 수도 있다.

```kotlin
class Controller(
    repository: Repository,
    logger: Logger
) : Repository by repository, Logger by logger
```

`Repository`를 구현할건데 `by` 뒤에 오는 인스턴스에 위임함으로써 구현하겠다고 해석하면 문법을 외우기 쉽다.

### 4. Objects, object expression & companion objects

**<u>object</u>**

`object`는 싱글톤이다. `object` 키워드를 사용함으로써 클래스 생성과 동시에 유일한 객체를 얻을 수 있다.

```kotlin
object KSingleton {
    fun foo() {}
}
```

자바에서 싱글톤을 만들기 위해 아래와 같은 코드를 작성해야 한다.

```java
public class JSingleton {
    public final static JSingleton INSTANCE = new JSingleton();
    
    private JSingleton() {}
    
    public void foo() {}
}
```

코틀린 `object`를 사용하면 위와 같은 코드가 생성된다.

**<u>object expression</u>**

Object expression은 자바의 익명 클래스에 대응된다. 인터페이스의 추상 메소드가 하나라면 익명 클래스가 아닌 람다를 사용할 수 있다.

```kotlin
window.addMouseListener(
    object: MouseAdapter() { // object expression
        override fun mouseClicked(e: MouseEvent) { ... }
        override fun mouseEntered(e: MouseEvent) { ... }
    }
)
```

Object expression은 싱글톤이 아니다. Object declaration(`object`)과는 다른 컨셉이다.

**<u>companion object</u>**

클래스에 inner class나 nested class 뿐만 아니라 nested object도 넣을 수 있다.
그 중 하나가 companion object이다. 이는 특별한 종류의 object로 클래스 이름을 통해 object의 멤버에 접근할 수 있다.

```kotlin
class A {
    companion object {
        fun foo() = 1
    }
}

fun main() {
    A.foo()
}
```

코틀린에는 `static` 메소드가 없다. Companion object가 `static` 메소드를 대체할 수 있다.

왜 `static` 대신 companion object를 두었을까? 어떤 경우에 companion object가 `static` 메소드보다 나을까.

companion object는 인터페이스를 구현할 수 있다.
`static` 메소드는 인터페이스의 멤버를 재정의할 수 없다.

```kotlin
class A {
    private constructor()
    
    companion object : Factory<A> {
        override fun create() = A()
    }
}

fun <T> createNewInstance(factory: Factory<T>)

// 클래스 이름을 통해 companion object 객체에 접근 가능
createNewInstance(A) // companion object가 Factory를 구현했으므로 인자로 전달 가능
        
A.create()

```

또한 companion object를 이용하여 extension을 정의할 수 있다. 클래스에 대한 extension과 companion object에 대한 extension을 구분하기 위해 후자를 정의할 때는 Companion이라는 접미사를 붙인다.

```kotlin
class Person(val firstName: String, val lastName) {
    companion object { ... }
}

fun Person.Companion.fromJSON(json: String): Person {
    ...
}

val p = Person.fromJSON(json) // 사용할 때는 클래스 이름만
```

코틀린에는 `static` 키워드가 없다.
- at the top-level
- inside objects
- inside companion objects

위의 위치에 `static` 멤버를 정의할 수 있다.
되도록 top-level에 정의하되, 클래스의 객체 또는 private 멤버에 접근해야 하는 경우와 같이 어쩔 수 없는 경우
objects 또는 companion objects 안에 정의하는 게 좋다.

기본적으로 objects 또는 companion objects에 정의된 멤버가 static 멤버로 컴파일되지는 않기 때문에 자바에서 static 멤버처럼 호출할 수 없다.
만약 자바에서 static 멤버처럼 호출하는 게 필요하다면 `@JvmStatic` 어노테이션을 붙여주어야 한다.

```kotlin
class C {
    companion object {
        @JvmStatic fun foo() {} // 자바에서 C.foo() ok
        fun bar() {} // 자바에서 C.bar() error, C.Companion.bar() ok
    }
}
```

object 역시 같다.

```kotlin 
object Obj {
  @JvmStatic fun foo() {} // 자바에서 Obj.foo() ok
  fun bar() {} // 자바에서 Obj.bar() error, Obj.INSTANCE.bar() ok
}
```

inner modifier는 오브젝트에 사용될 수 없다.
오브젝트는 싱글톤이므로 하나의 객체만 존재하는데, 만약 오브젝트가 inner modifier와 함께 사용될 경우 outer 클래스의 객체가 여러개 존재할 수 있는 셈이고,
이럴 경우 오브젝트가 outer 클래스 객체의 어떤 레퍼런스를 가지고 있어야 하는지가 명확하지 않기 때문이다. Nested object는 문제 없다.

### 5. Constants

`const`: for primitive type and String. make compile-time constant

코틀린에서는 프로퍼티에 `const`를 붙이면 컴파일타임 상수가 된다. 컴파일 시점에 `const`가 붙은 프로퍼티는 모두 실제값으로 대체된다.

`The Kotlin compiler also inlines the value of such contant. It replaces its name with its value everywhere in the code.`

`@JvmField`: eliminates accessors

코틀린의 프로퍼티를 자바의 필드로 노출한다. Top-level 또는 오브젝트의 안에 사용될 경우 static 필드를 생성하고, 일반 클래스 안에서 사용될 경우 일반 필드를 생성한다.

### 6. Generics

코틀린에서는 제네릭 매개변수를 사용하는 함수를 다음과 같이 정의한다.

`fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>`

```kotlin
fun <T> foo(list: List<T>) {
    for (element in list) { // element can be null
        ...
    }
}
```

위와 같이 정의하면 제네릭 매개변수가 nullable 타입인지 non-nullable 타입인지 알 수 없다.
타입 매개변수를 non-nullable 타입으로 제한하고 싶다면 아래와 같이 non-null upper bound를 사용한다.

```kotlin
fun <T : Any> foo(list: List<T>) {
    for (element in list) {
        ...
    }
}
```

자바와 마찬가지로 upper bound에서 역시 타입 매개변수를 가질 수 있다.

```kotlin
fun <T : Comparable<T>> max(first: T, second: T): T {
    return if (first > second) first else second
}
```

타입 매개변수에 여러 제약을 주고 싶을 때는 where 키워드를 사용한다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T) 
        where T : CharSequence, T : Appendable {
    if (!seq.endsWith('.')) {
        seq.append('.')
    }
}
```

### 7. OOP design choices

`Why final and public are used by default?`

가장 제약이 강한 final과 가장 제약이 약한 public을 디폴트로 삼은 것은 애플리케이션 개발자와 라이브러리 개발자의 니즈를 모두 만족하기 위함이다.
일반적으로 public은 애플리케이션 코드를 위한 최선의 선택이고, final은 라이브러리 코드를 위한 최선의 선택이다.

### 8. TMI

**<u>Equals & reference equality</u>**

- `==` calls equals
- `===` checks reference equality

**<u>Same JVM signature</u>**

```kotlin
fun List<Int>.average(): Double { ... }
fun List<Double>.average(): Double { ... }
```

JVM 시그니처가 같은 두 개의 함수를 정의할 수 없다.
JVM 시그니처에는 제네릭 타입 매개변수가 포함되지 않으므로 위의 두 함수는 같은 시그니처를 갖는다.
따라서 함수 오버로딩을 의도했으나 컴파일 에러가 발생한다.

코틀린에서는 `@JvmName`이라는 어노테이션을 통해 위와 같이 JVM 시그니처가 같은 두 함수를 정의할 수 있다.

```kotlin
fun List<Int>.average(): Double { ... }
@JvmName("averageOfDouble")
fun List<Double>.average(): Double { ... }
```

위의 두 함수는 바이트코드 레벨에서 이름이 다르다.

코틀린에서는 마치 함수 오버로딩이 적용된 것처럼 `average()`를 통해 위의 두 함수를 호출할 수 있으나,
자바에서는 코틀린을 바라볼 때 컴파일된 결과를 통해서만 바라볼 수 있기 때문에
두 번째 함수를 호출할 때는 `averageOfDouble()`를 통해 호출해야 한다.
