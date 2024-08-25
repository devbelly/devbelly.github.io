---
title: Dispatchers.IO 와 limitedParallelism
date: 2024-08-24T10:51:23+09:00
draft: false
comments: true
toc: false
tags:
  - study
---
안녕하세요 오늘은 코루틴을 사용하며 겪었던 메모리 부족 문제를 공유하고자 합니다.

원하는 정보를 S3에 업로드 하기 위해 아래처럼 코드를 작성했습니다.

```kotlin
fun main() {  
    // @Async  
    thread {  
        uploadImage()  
    }  
}  
  
fun uploadImage() {  
    CoroutineScope(Dispatchers.IO).launch {  
        awsS3Upload()  
    }  
}  
  
suspend fun awsS3Upload() {  
    // AWS S3 Upload  
    // with(Dispatchers.Default){
    //     some work...
    // }
}
```

쓰레드마다  `uploadImage()`를 호출하고 그 안에서는 각자 코루틴 빌더 `launch`를 사용합니다. 코루틴 빌더로 코루틴을 생성했다면 쓰레드와 연결 해야합니다. 이 작업은 CoroutineDispatcher가 담당합니다.

<br>

## OOM 발생 

```
Running on thread: DefaultDispatcher-worker-9  
Running on thread: DefaultDispatcher-worker-65  
## OOM 발생! something wrong...
Running on thread: DefaultDispatcher-worker-52  
Running on thread: DefaultDispatcher-worker-20  
Running on thread: DefaultDispatcher-worker-29
```

올바르게 동작할 것이라 생각하고 코드를 실행하면 간헐적으로 OOM이 발생했습니다. OOM이 발생했을 때 특징 중 한 가지는 워커 쓰레드의 번호가 커지면 OOM이 발생했다는 점이였습니다. 

<br>

## 예상 원인 1, Unstructured Concurrency

코드에서 두 가지 원인이 있을 것이라고 생각했습니다. 첫 번째는 `Unstructured Concurrency` 입니다. [코틀린 공식 가이드](https://kotlinlang.org/docs/coroutines-basics.html#structured-concurrency)에서는 `structured concurrency` 형태를 권장하는데, 무슨 뜻일까요? 

> Structured concurrency ensures that they are not lost and do not leak. An outer scope cannot complete until all its children coroutines complete. Structured concurrency also ensures that any errors in the code are properly reported and are never lost.

해석하자면, 부모 코루틴은 자식 코루틴들이 모두 수행될 때까지 종료되지 않고 error를 확인할 수 있다고 합니다. 

`uploadImage()`에서는 `CoroutineScope`로 새로운 스코프를 정의합니다. 만약 `awsS3Upload()` 내부에서 `CoroutineScope` 처럼 새로운 코루틴 스코프를 정의한다면 `unstructured` 형태가 됩니다. (`coroutineScope`과 `CoroutineScope`는 다릅니다!)

<br>

<img width="920" alt="image" src="https://github.com/user-attachments/assets/f9e5e6c1-e8be-4d8c-9f18-6e17e6e5d681">

[스택오버플로우](https://stackoverflow.com/questions/59368838/difference-between-coroutinescope-and-coroutinescope-in-kotlin)에서도 메모리 누수를 가져올 수 있어 지양하라고 설명합니다. 하지만 진행한 프로젝트에서는 따로 자식 코루틴을 생성하지 않아 이 경우는 아니라고 판단했습니다.

<br>

## 예상 원인 2, Dispatchers.IO

<img width="796" alt="image" src="https://github.com/user-attachments/assets/0ad336b5-15fb-460c-87a8-59fda8439266">

Dispatcher에는 대표적으로 Main, IO, Default가 있습니다. 그 중에서 `Dispatchers.IO`는 네트워크/DB 입출력이 있는 작업들에 대해 코루틴을 적절한 Thread로 할당하는 역할을 합니다. 즉, 우리가 CoroutineDispatcher에 코루틴을 보내기만 하면, CoroutineDispatcher은 자신이 사용할 수 있는 스레드가 있을 때 코루틴을 **스레드로** 보내 실행시킵니다.

<br>

```kotlin
fun main() = runBlocking {  
    repeat(1000) {  
        launch(Dispatchers.IO) {  
            Thread.sleep(200)  
  
            val threadName = Thread.currentThread().name  
            println("Running on thread: $threadName")  
        }  
    }
}
/**
Running on thread: DefaultDispatcher-worker-38
Running on thread: DefaultDispatcher-worker-27
Running on thread: DefaultDispatcher-worker-68
Running on thread: DefaultDispatcher-worker-12
Running on thread: DefaultDispatcher-worker-60
Running on thread: DefaultDispatcher-worker-58
...
*/
```

`Dispatchers.IO`는 쓰레드 풀에서 최대 64개의 쓰레드를 사용하도록 제한되어있습니다. 64보다 큰 숫자가 보이는 이유는 `Default`와 `IO`가 쓰레드 풀을 공유하기 때문입니다. `Default`는 자신의 코어 갯수만큼 쓰레드를 사용하고 `IO`는 최대 64개를 사용하므로 쓰레드 풀에는 대략 76개의 쓰레드가 있습니다. 다만, `IO`에서 동시에 사용하는 쓰레드는 최대 64개 인 것은 변하지 않습니다.

OOM이 발생하는 상황은 항상 `Dispatchers.IO`에서 동시에 사용되는 쓰레드의 수가 많아지면 발생했습니다. 동시에 실행되는 쓰레드 수를 줄여야겠다고 판단했습니다. 코틀린에서는 [limited-parallelism](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/limited-parallelism.html)을 통해 디스패처가 사용하는 쓰레드 수를 제한할 수 있습니다.

```kotlin
fun main() = runBlocking {  
    repeat(1000) {  
        launch(Dispatchers.IO) { 
            println("IO  : running in thread ${Thread.currentThread().name}")  
        }  
    }
}
```

```
IO  : running in thread DefaultDispatcher-worker-64 @coroutine#856
IO  : running in thread DefaultDispatcher-worker-21 @coroutine#857
IO  : running in thread DefaultDispatcher-worker-64 @coroutine#858
IO  : running in thread DefaultDispatcher-worker-21 @coroutine#859
IO  : running in thread DefaultDispatcher-worker-66 @coroutine#750
```


```kotlin
fun main() = runBlocking {  
    val dispatcher = Dispatchers.IO.limitedParallelism(12)  
    repeat(1000) {  
        launch(dispatcher) { // will get dispatched to DefaultDispatcher  
            println("Default  : running in thread ${Thread.currentThread().name}")  
        }  
    }
}
```

```
IO  : running in thread DefaultDispatcher-worker-1 @coroutine#906
IO  : running in thread DefaultDispatcher-worker-1 @coroutine#940
IO  : running in thread DefaultDispatcher-worker-21 @coroutine#901
IO  : running in thread DefaultDispatcher-worker-15 @coroutine#900
IO  : running in thread DefaultDispatcher-worker-18 @coroutine#898
IO  : running in thread DefaultDispatcher-worker-15 @coroutine#943
```

따라서 `limitedParallelism`을 사용해 동시에 실행되는 쓰레드 수를 12개로 제한하니 OOM 문제를 해결할 수 있었습니다. 

<br>

## 참고

- [Dispatchers — Kotlin Coroutines](https://medium.com/@wind.orca.pe/dispatchers-kotlin-coroutines-659a5681f329)
- [Coroutine, Structured Concurrency](https://jowunnal.github.io/coroutines/android_structured_concurrency/)
- [조세영의 Kotlin World](https://kotlinworld.com/)
- [Coroutine IO Dispatcher의 Thread number가 최대 Thread 갯수를 초과하는 이슈](https://sandn.tistory.com/112)




