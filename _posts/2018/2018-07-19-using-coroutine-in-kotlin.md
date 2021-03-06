---
layout: post
title: "Using coroutine in Kotlin"
author:
published: true
modified:
categories: kotlin
excerpt: Coroutine 에 대한 소개
tags: [coroutine]
image:
  feature:
date: 2018-7-19
---

## Kotlin coroutine
`kotlinx.coroutines` 에는 다양한 표현들이 있습니다. `launch` 또는 `async` 등등의 코루틴의 핵심 요소들이 있습니다. 이러한 표현들을 사용하여 코루틴을 어떻게 프로젝트에서 사용하는지 소개 합니다.

<br>
#### Coroutine basics
코루틴의 기본적인 컨셉입니다.

``` kotlin
fun main(args: Array<String>) {
    launch { // launch new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
```
launch 를 이용하여 코루틴을 시작 합니다. launch 로 넘겨준 블록은 백그라운드에서 실행이 됩니다. 그리고 기본적으로는 블록안의 코드는 순차적으로 실행이 됩니다. 그래서 위의 코드의 실행 결과를 살펴보면 
```
Hello,
World!
```

코루틴은 `light-weight threads` 경량화된 쓰레드 이고, `launch` 같은 애들을 `coroutine builder`라 부르며 코루틴은 해당 키워드로 실행이 합니다. 또한 코루틴의 블록 안에는 `suspend` 키워드가 붙은 함수만을 사용해야 합니다. 또한 `launch`는 `job` 을 반환하는데 이 객체를 통해 `cancel` 을 할수 있고, 또는 `join` 함수를 통해 `job`이 끝날때 까지 기다릴수 있습니다. 

suspend function 은 스레드와 스케쥴의 관리를 수행하는 것 이 아닌, 비동기 실행을 위한 중단 지점을 정의 하며, 코루틴은 함수를 중단하고 다시 실행 함으로써 작업을 효율적으로 처리 하는 개념 입니다. 

``` kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch { doWorld() }
    println("Hello,")
    job.join()
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

<br>
### Using coroutine
이러한 점들을 활용하여 일반적으로 많이 사용될 만한 흐름을 좀더 깔끔하고 간결하게 코드를 작성할수 있게 됩니다. 조금 더 예시를 살펴 보면, 우선은 단순히 동기적으로 코드를 나열하면 아래와 같은 코드가 됩니다. 

``` kotlin
fun requestToken(): Token { ... }
fun createPost(token: Token, item: Item): Post { ... }
fun updateUI(post: Post) { ... }

fun postItem(item: Item) {
  val token = requestToken()
  val post = creatPost(token, item)
  updateUI(post)
}
```

위와 같은 코드로 그대로 실행한다면 UI 쓰레드에서 긴 시간의 작업을 하게 되며 UI가 끊기는 현상을 볼수 있게 될 것 입니다. 그것을 해결하기 위하여 보통 가장 많이 이용하는 방법이 백그라운드 쓰레드로 처리한후 callback 으로 알려주는 방식 입니다. 코드는 아래와 같이 됩니다.

``` kotlin
fun requestTokenAsync(cb: (Token) -> Unit) { ... }
fun createPostAsync(token: Token, item: Item, cb: (Post) -> Unit) { ... }
fun updateUI(post: Post) { ... }

fun postItem(item: Item) {
  requestTokenAsync { token ->
    creatPostAsync(token, item) { post ->
      updateUI(post)
    }
  }
}
```

위 코드를 coroutine 을 이용하여 다시 리펙토링 해보면, 아래와 같이 동기식 코드로 짤 수 있습니다.

``` kotlin
suspend fun requestToken(): Token {
  return token
}

suspend fun createPos(token: Token, item: Item): Post {
  return post
}

suspend fun postItem(item: Item) {
  val token = requestToken()
  val post = createPost(token, item)
  processPost(post)
}
```

이 코드를 실제로 사용하려고 하면, suspend 함수는 suspend 함수 안에서만 사용할 수 있기 때문에, postItem 을 바로 호출을 할 수 없습니다. 이 때 `Coroutine Builder` 를 통해 코루틴을 실행 할수 있는 환경을 만들어 주게 됩니다. 코드를 통해 살펴 보면, 아래와 같이 `launch` 라는 builder 를 이용하여 구현 할 수 있습니다.

``` kotlin
fun postItem(item: Item) {
  launch {
    val token = requestToken()
    val post = createPost(token, item)
    processPost(post)
  }
}
```

이 때 `launch` 의 파라미터로 `CoroutineContext` 를 넘길수가 있는데, 이 Context 를 이용하여 해당 코루틴이 어떤 쓰레드에서 실행이 되도록 할지 지정할 수 있습니다. `launch(UI) { ... }` 와 같이 실행을 하면 해당 launch 안의 코루틴은 UI 쓰레드에서 동작하게 됩니다.


<br>
[참고] <br>
[Generator 코루틴 / Async & Await 코루틴](https://medium.com/@jooyunghan/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EC%86%8C%EA%B0%9C-504cecc89407)<br>
[Kotlin 코루틴은 어떻게 동작하는가? - 도창욱](https://www.youtube.com/watch?v=usaD7HyN598)<br>
[Kotlin Conf 2017 - Introduction to Coroutine](https://www.youtube.com/watch?list=PLQ176FUIyIUY6UK1cgVsbdPYA3X5WLam5&v=_hfBv0a09Jc)<br>
