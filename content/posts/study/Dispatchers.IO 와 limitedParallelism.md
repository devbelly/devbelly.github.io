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

## 예상 원인 2, Continuation
<img width="796" alt="image" src="https://github.com/user-attachments/assets/0ad336b5-15fb-460c-87a8-59fda8439266">

Dispatcher에는 대표적으로 Main, IO, Default가 있습니다. 그 중에서 `Dispatchers.IO`는 네트워크/DB 입출력이 있는 작업들에 대해 코루틴을 적절한 Thread로 할당하는 역할을 합니다. 즉, 우리가 CoroutineDispatcher에 코루틴을 보내기만 하면, CoroutineDispatcher은 자신이 사용할 수 있는 스레드가 있을 때 코루틴을 **스레드로** 보내 실행시킵니다.

<br>

그렇다면 코루틴은 어디에 저장되어있을까요? 아래 코드를 실행한 뒤, 힙 덤프를 분석해 보겠습니다.

```kotlin
fun main() = runBlocking {  
    repeat(100000) {  
        launch(Dispatchers.IO) {  
            Thread.sleep(200)  
            withContext(Dispatchers.IO) {  
                println("withContext Running on thread: ${Thread.currentThread().name}")  
            }  
  
            val threadName = Thread.currentThread().name  
            println("Running on thread: $threadName")  
        }  
    }}
```

<img width="514" alt="image" src="https://github.com/user-attachments/assets/af165c51-7790-4776-9a0f-a3c8230121aa" />

분석 파일을 보면 `LimitedDispatcher` 이 `LockFreeTaskQueue`를 갖고 있고 해당 큐에는 `Continuation`이 저장되어있습니다. `Continuation`은 우리가 함수를 일시중단 했을 때 어디서부터 다시 실행할지 정보를 저장한 객체입니다. 정리하자면 코틀린은 우리가 작성한 코루틴을 CPS 스타일로 변경을 하고 `Continuation` 객체를 통해 하나의 스레드에서도 여러 코루틴을 실행가능하게 합니다. 

<br>

<img width="909" alt="image" src="https://github.com/user-attachments/assets/65258309-5659-4c9a-8a6d-162d42bf7b97" />

`LimitedDispatchers`는 `parallelism` 만큼 워커 노드를 실행하는 것을 알 수 있습니다. [자체적인 큐](https://github.com/Kotlin/kotlinx.coroutines/blob/fed40ad1f9942d1b16be872cc555e08f965cf881/kotlinx-coroutines-core/common/src/internal/LockFreeTaskQueue.kt#L71)는 최대 $2^{30}$ 늘어날 수 있습니다.  따라서 예상할 수 있는 문제는 다음과 같습니다.

> 너무 많은 코루틴이 생성되면 Heap에 Continuation 객체가 많이 저장되어 OOM 가능성이 있다.

이러한 문제를 해결하기 위해 동시에 실행되는 쓰레드 수를 줄여야겠다고 판단했습니다. 코틀린에서 제공하는 [limited-parallelism](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/limited-parallelism.html)을 통해 디스패처가 사용하는 쓰레드 수를 제한하여 문제를 해결할 수 있었습니다.

<br>

## 참고

- [Dispatchers — Kotlin Coroutines](https://medium.com/@wind.orca.pe/dispatchers-kotlin-coroutines-659a5681f329)
- [Coroutine, Structured Concurrency](https://jowunnal.github.io/coroutines/android_structured_concurrency/)
- [조세영의 Kotlin World](https://kotlinworld.com/)
- [Coroutine IO Dispatcher의 Thread number가 최대 Thread 갯수를 초과하는 이슈](https://sandn.tistory.com/112)
- [빙티의 코틀린 코루틴 동작방식](https://www.youtube.com/watch?v=iy-ouIRGDOY)




