---
layout: post
title: "Kotlin - inline function"
author:
published: true
modified:
categories: kotlin
excerpt: Kotlin 의 inline function 에 대해서 자세히 알아봅니다.
tags: [android, kotlin, inline function]
image:
  feature:
date: 2017-7-19
---

### Inline function
>Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure, i.e. those variables that are accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.

>But it appears that in many cases this kind of overhead can be eliminated by inlining the lambda expressions. The functions shown below are good examples of this situation. I.e., the lock() function could be easily inlined at call-sites. Consider the following case:

<br>
위의 내용이 코틀린 공식 사이트 레퍼런스 가이드에 나와있는 내용이다. 내용을 살펴보면,
고차 함수를 사용하면 런타임에 패널티가 있고, 각 함수는 객체이고, 클로저(함수에서 액세스되는 변수)를 가지고 있다. 따라서 메모리를 차지하게 되고, 함수 콜을 하기위한 런타임 오버헤드가 있다.

하지만 함수 구현자체를 코드내부에 넣어버림으로써(함수를 실행하는 지점에 코드를 넣음) 이러한 오버헤드를 없앨수 있다.

아래 lock() 함수의 예시를 보자.

``` java
lock(l) { foo() }

fun lock(l: Any, work: () -> Unit)  {
    l.lock()
    try {
      work()
    } finally {
      l.unlock()
    }
}
```

위의 코드를 보면 `lock(l) { foo() }` 이렇게 함수를 호출하게 되면, `{ foo() }` 자체도 객체이기 때문에 메모리에 올라가게 되고, `lock()` 함수를 호출할 때 호출 비용이 들게 된다.

그래서 inline 키워드를 써서 inline 함수로 만들게 되면, 컴파일러에서 코드를 생성하여 집어 넣게 된다. 그러니까 아래와 같이 된다는 말이다.
``` java
inline fun lock(l: Any, work: () -> Unit)  {
    l.lock()
    try {
        work()
    } finally {
        l.unlock()
    }
}

lock(l) { foo() }  // 이 코드를 컴파일러가 아래와 같이 바꿔준다

l.lock()
try {
    foo()
} finally {
    l.unlock()
}

```

다만 컴파일러가 작업을 해줘야 하기때문에 성능에 문제가 생길 수 있다고 적혀있고, 따라서 코드가 긴함수를 넣지 말라고 적혀 있다. 여기서 약간 의문점이, 컴파일단에서 코드를 제너레이트 해서 넣는다면 런타임 성능에는 큰 문제가 없는것이 아닌가?? 라는 의문이다.


<br>
<br>
(+ reified 키워드도 연달에 포스팅 할 예정)
<br>
<br>
