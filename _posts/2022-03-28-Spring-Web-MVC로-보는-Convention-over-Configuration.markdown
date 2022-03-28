---
layout: post
title:  "Spring Web MVC로 보는 Convention over Configuration"
date:   2022-03-28 22:00:00 +0900
categories: TIL
---

### Convention over Configuration
`Convention over Configuration(이하 CoC)`이란 convention을 이용함으로써
개발자가 내려야 하는 의사결정을 줄여주는 소프트웨어 디자인 패러다임이다.

예를 들어, `Hibernate`라는 프레임워크는 `Product`라는 도메인 클래스를 만들면
테이블 이름을 설정하지 않았을 때 이에 대응되는 DB 테이블을 `Product`로 이름짓는다.

`CoC`가 적용된 프레임워크를 사용하는 개발자는 애플리케이션의 unconventional한 부분만 명시해주면 된다.

Spring Web MVC에 적용된 `CoC` 사례를 살펴보자.

### DefaultRequestToViewNameTranslator
Spring Web MVC의 `DefaultRequestToViewNameTranslator` 클래스는 `RequestToViewNameTranslator`의 구현체이다.
`RequestToViewNameTranslator`는 단순히 들어온 요청 URI를 뷰 이름으로 변환하는 일을 한다.
`DefaultRequestToViewNameTranslator`는 요청 URI에 붙어 있는 파일의 확장자와 앞에 오는 `/`,
뒤에 오는 `/`를 자르고 설정된 prefix와 suffix가 있을 경우 이들을 붙여 뷰 이름으로 반환한다.
(`stripLeadingSlash`와 `stripExtension` 프로퍼티를 이용하여
앞에 오는 `/`와 파일 확장자를 자르지 않도록 각각 설정할 수 있다.)

```
http://localhost:8080/gamecast/display.html » display
http://localhost:8080/gamecast/displayShoppingCart.html » displayShoppingCart
http://localhost:8080/gamecast/admin/index.html » admin/index
```

### Model 
Spring Web MVC의 `Model` 클래스는 model attributes를 저장하기 위한 홀더이다.
`Map`처럼 사용할 수 있는 `Model`에는 다음과 같은 메소드가 있다.
```java
Model addAttribute(String attributeName, @Nullable Object attributeValue);
Model addAttribute(Object attributeValue);
```
아래 메소드의 경우 attribute를 설정하기 위해 필수적인 값인 `attributeName`이 없는데
`org.springframework.core.Conventions`의 `getVariableName()` 메소드를 이용하여
convention name을 `attributeName`으로 삼은 뒤 attribute를 추가한다.
다음은 `getVariableName()`의 구현이다.

```java
public static String getVariableName(Object value) {
    Assert.notNull(value, "Value must not be null");
    Class<?> valueClass;
    boolean pluralize = false;

    if (value.getClass().isArray()) {...}
    else if (value instanceof Collection) {...}
    else {
        valueClass = getClassForValue(value);
    }

    String name = ClassUtils.getShortNameAsProperty(valueClass);
    return (pluralize ? pluralize(name) : name);
}
```

`CoC`에 집중하기 위해 배열도 컬렉션도 아닌 경우만 보자면,
해당 클래스의 이름을 반환하는 것을 확인할 수 있다.
다시 말해, `Model`의 인자가 하나인 `addAttribute()`는
주어진 `attributeValue`의 클래스 이름(convention name)을 `attributeName` 삼는다.


### 레퍼런스
[Convention over configuration - Spring Web MVC framework 레퍼런스](https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch16s10.html)
<br>[Convention over configuration - 위키백과](https://en.wikipedia.org/wiki/Convention_over_configuration)