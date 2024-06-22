---
title: "쓰레드와 @Async"
date: 2023-01-11T11:39:38+09:00
draft: false
comments: true
toc: false
tags:
  - spring
---

## Thread Pool

하나의 프로세스에서 여러 개의 쓰레드를 사용함으로써 동시에 작업을 처리할 수 있습니다. 하지만 단순하게 여러 개의 쓰레드를 사용한다고 해서 효율적으로 작업을 처리할 수 있는 것은 아닙니다.

예를 들어 매 요청마다 쓰레드를 생성해서 작업을 처리한다면 어떻게 될까요? 쓰레드는 크게 커널 수준 쓰레드와 사용자 수준 쓰레드로 나눌 수 있는데 자바에서는 사용자 수준 쓰레드 방식을 채택해 사용합니다. 일반적으로 사용자 수준 쓰레드 방식은 하나의 커널 쓰레드에 여러개의 사용자 쓰레드를 매핑하는 방식을 사용합니다. 이 경우 하나의 쓰레드가 `block` 되면 다른 쓰레드가 중단되는 문제가 발생합니다. **하지만** Java에서는 `One to One Threading Model`로 쓰레드를 생성함으로써 이러한 문제를 극복했습니다. 하나의 쓰레드가 중단되어도 다른 쓰레드에 영향을 미치지 않게 됩니다.

![image](https://user-images.githubusercontent.com/67682840/211701888-64a2766e-1496-4fb1-8107-1bc660702654.png)

> **Green Thread?** <br>
> 자바 1.2버전 이전까지는 JVM이 쓰레드 스케쥴링을 했지만 이후로는 OS 정책에 맡기도록 변경되었습니다. OS 대신 런타임 라이브러리나 VM의해 관리되는 쓰레드 스케쥴링 기법을 Green thread라고 부릅니다. 이 방식에서 blocking system call을 호출하면 프로세스의 모든 thread가 block 당하게 됩니다.

이는 user Thread 생성시 kernel Thread와 연결해야한다는 의미이고 kernel에 접근하여 작업이 필요하다는 것을 의미합니다. 커널에 많이 접근할수록 성능저하가 발생하므로 쓰레드 생성에는 많은 비용이 필요함을 알 수 있습니다. 즉, 접근마다 쓰레드를 생성하면 발생할 수 있는 첫번째 문제점은 쓰레드 생성, 삭제에는 많은 비용이 필요하다는 점입니다.

두 번째 문제점은 처리속도보다 많은 요청이 들어오게 되면 쓰레드 유지에 많은 메모리를 사용하고 컨텍스트 스위칭이 더 자주 발생한다는 것입니다.

이러한 문제점을 해결하기 위해 자바에서는 쓰레드 풀을 사용합니다. 쓰레드 풀이란 쓰레드를 허용된 개수 안에서 사용하도록 제한하는 시스템입니다.

![image](https://user-images.githubusercontent.com/67682840/211702758-44c60df9-6351-4465-a6cf-96161de98096.png)

쓰레드 풀은 요청이 들어오면 해당 요청을 저장할 작업큐와 작업큐에서 해당 요청들을 꺼내 처리할 쓰레드들로 이루어져 있습니다. 작업이 완료되면 쓰레드를 삭제하는 대신 쓰레드풀에 쓰레드를 반납하므로 위에서 언급한 쓰레드 생성시 많은 비용이 발생하는 문제를 해결할 수 있고 최대 쓰레드 생성갯수를 제한하므로 두 번째 문제점 또한 해결가능합니다.

자바에서는 ThreadPoolExecutor를 이용해 쓰레드풀을 구현하고 아래와 같은 옵션을 통해 어플리케이션에 맞는 설정을 할 수 있습니다.

- maximum-pool-size: 최대 쓰레드 개수
- core-pool-size: 최소 쓰레드 개수
- keep-alive-time: 해당 시간 이후로도 요청이 없다면 최소 쓰레드만큼 유지하고 쓰레드가 삭제된다.

<br>

자바에서 사용하는 쓰레드 풀을 기반으로 톰캣에서 사용하는 쓰레드 풀에 대해서 알아봅시다. 톰캣에서는 `org.apache.tomcat.util.threads.ThreadPoolExecutor`에 선언된 `ThreadPoolExecutor`을 사용합니다. 자바와 거의 유사하나 위에서 언급한 특징 외에 두 가지를 더 알아야합니다.

![image](https://user-images.githubusercontent.com/67682840/211704109-3bf35aa2-e9d0-4d9a-8372-4f0c1cfb450b.png)

- max-connections: 톰캣이 최대로 동시에 처리할 수 있는 connection의 갯수
- accept-count: max-connections 이상의 요청이 들어왔을 때 사용하는 queue의 크기

위 내용을 공부하면서 `max-connections`와 `maximum-pool-size`는 무슨 관계가 있는지 궁금했습니다. 이를 이해하기 위해서는 톰캣의 구성요소중 connector의 역할에 대해 알아야합니다. connector은 특정 TCP port에서 request를 listen해 engine으로 전달하는 역할을 합니다. (engine은 tomcat의 구성요소중 하나) Connector의 버전마다 동작하는 방식이 다른데 크게 `BIO` 방식과 `NIO`방식으로 나눌 수 있습니다.

- `BIO`: Tomcat 7의 기본방식, 하나의 thread가 하나의 connection을 담당
- `NIO`: Tomcat 8.5부터 기본방식, 하나의 thread가 여러 connection을 담당

이러한 이유로 Non-blocking 방식을 사용하는 톰캣의 connector 덕분에 Thread Pool의 최대 스레드 갯수보다 많은 양의 connection을 유지할 수 있습니다.

<br>

#### SimpleAsyncTaskExecutor

톰캣의 쓰레드 풀에 대해 공부했으니 이제 스프링에서 비동기작업을 구현하는 방법에 대해 알아보겠습니다. 스프링에서 `@EnableAsync`를 사용하면 간단하게 비동기 처리를 할 수 있습니다. 해당 선언을 하게 되면 쓰레드 풀과 관련된 정의를 찾기 시작합니다. 대표적으로

- `org.springframework.core.task.TaskExecutor`
- `java.util.concurrent.Executor`

이 해당합니다. 만약 이들 중 하나라도 찾지 못하면 비동기 메서드 호출을 위해 `org.springframework.core.task.SimpleAsyncTaskExecutor`을 사용하게 되는데 해당 클래스는 **어떠한 쓰레드도 재사용하지 않고** 호출마다 **새로운 쓰레드를 시작**합니다.

위 방식은 자원낭비가 심각하므로 `AsyncConfigurer`을 구현한 클래스를 정의해서 사용자 정의 쓰레드 풀을 생성할 수 있습니다.

<br>

#### ThreadPoolTaskExecutor

사용자 정의 쓰레드 풀을 위해 `ThreadPoolTaskExecutor`을 사용했습니다. 어떠한 설정을 제공하는지 같이 살펴보도록 하겠습니다.

![image](https://user-images.githubusercontent.com/67682840/211744539-a7dfdbc0-cb82-4155-8d5b-2c0fa430ef05.png)

**capacity**

```java
executor.setCorePoolSize(10); // 기본 쓰레드 수
executor.setMaxPoolSize(50); // 최대 쓰레드 수
executor.setQueueCapacity(100); // Queue 사이즈
```

위와 같이 설정한다면 10개의 쓰레드에서 처리하다가 처리 속도가 밀릴 경우 100개 만큼 큐에서 대기하고 이보다 더 많은 요청이 들어올 시 50개의 쓰레드를 생성해 작업을 처리하게 됩니다.

**RejectedExecutionHandler**

max 쓰레드까지 생성하고 queue까지 꽉 찬 상태에서 추가요청이 오면 `RejectedExecutionException`이 발생합니다. 스프링에서는 해당 예외를 핸들링할 수 있는 몇가지 옵션을 제공합니다.

- **AbortPolicy**
  - 기본설정
  - RejectedExecutionException이 발생
- **DiscardOldestPolicy**
  - 오래된 작업을 skip
  - 모든 task가 무조건 처리될 필요가 없는 경우 사용
- **DiscardPolicy**
  - 처리하려는 작업을 skip
  - 모든 task가 무조건 처리될 필요가 없는 경우 사용
- **CallerRunPolicy**
  - shutdown가 아니라면 ThreadPoolTaskExecutor에 요청한 쓰레드에서 직접 처리

**shutdown**

쓰레드 풀에서 작업중인데 어플리케이션 종료 요청이 온다면 아직 수행할 task가 남아있음에도 즉시 종료됩니다. 이렇게 되면 아직 처리하지 못한 task가 유실되므로 이를 방지하기 위해 `setWaitForToCompleteOnShutdown()` 을 제공합니다.

**Timeout**

만약 모든 작업이 처리되길 기다리기 어렵다면 `setAwaitTerminationSeconds()`를 통해 최대 종료 대기 시간을 설정할 수 있습니다.

<br>

#### @Async

위 설정이 끝났다면 비동기적으로 실행할 메서드에 `@Async`를 추가하면 됩니다. 주의할 점은 해당 어노테이션의 리턴값은 `void` 나 `Future`을 가지므로 요청결과를 받아오기 위해선 `ListenableFuture`이나 `CompletableFuture`을 사용해야합니다.

![image](https://user-images.githubusercontent.com/67682840/211987385-35034333-8515-48cc-b21c-075a4e099788.png)

`CompletableFuture`에 관한 내용은 [이글](https://brunch.co.kr/@springboot/267)에 자세히 정리되어있으므로 꼭 읽어보시기 바랍니다.

`@Async`는 스프링 AOP를 통해 구현되어있습니다. 어노테이션을 살펴보면 마지막에 리턴타입에 따라 `Async`가 어떻게 구현되어있는지 살펴볼 수 있는데 자세한 내용은 [이 글](https://brunch.co.kr/@springboot/401)을 살펴보면 좋습니다.

<br>

#### Reference

- https://velog.io/@sunaookamisiroko/%EC%9E%90%EB%B0%94%EC%9D%98-%EC%93%B0%EB%A0%88%EB%93%9C%EB%8A%94-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%88%98%EC%A4%80-%EC%93%B0%EB%A0%88%EB%93%9C%EC%9D%B8%EA%B0%80
- https://www.youtube.com/watch?v=um4rYmQIeRE
- https://stackoverflow.com/questions/57988341/what-are-the-defaults-in-spring-async
- https://kapentaz.github.io/spring/Spring-ThreadPoolTaskExecutor-%EC%84%A4%EC%A0%95/#
- https://brunch.co.kr/@springboot
