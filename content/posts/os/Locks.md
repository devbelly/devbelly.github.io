---
title: Locks
date: 2024-05-12T11:36:55+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
동시 프로그래밍의 근본적인 문제 중 하나는 일련의 명령어를 원자적으로 실행하고 싶지만 단일 프로세서에서 인터럽트를 걸기 때문에 실행할 수 없다는 점입니다. 따라서 이번 시간에는 Lock을 통해 어떻게 이 문제를 해결할 수 있는지 살펴보겠습니다

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9c513cb4-3c64-4a18-9157-d40837626685">

Lock을 사용하려면 간단하게 변수를 선언하면 됩니다. 기본적으로 Lock은 획득 또는 잠금 상태가 존재합니다. 하나의 쓰레드가 `lock()`을 호출하면 잠금을 획득하려고 시도하고, 다른 쓰레드가 Lock을 갖고 있지 않다면 해당 쓰레드가 Lock을 획득하고 `critical section`에 진입합니다. 다른 쓰레드가 동일한 Lock에 대해 `lock()`을 호출하면 첫 번째  쓰레드가 Lock을 소유하는 동안 반환되지 않으므로 `critical section`은 하나의  쓰레드만 실행됨을 보장할 수 있습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ae3b22b5-9da9-40ec-abde-b2592bd1042c">

POSIX library에서 제공하는 Lock을 `Mutex`라고 합니다. `Mutex`의 `lock()`과 `unlock()`의 동작에 대해서는 뒤에서 자세하게 다루겠습니다. 일단은 쓰레드간 상호 배제를 위해 사용하는 Lock인 것만 알아둡시다. `lock()`과 `unlock()`시 변수를 전달하는데 서로 다른 변수를 보호하기 위해 서로 다른 Lock을 사용할 수 있기 때문입니다. 하나의 큰 Lock을 사용하는 대신 세분화된 Lock을 사용한다면 여러 쓰레드가 접근해 동시성을 높일 수 있습니다.

## Evaluating Locks

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/94ec6c56-2307-403a-bf41-94180ae8d297">

효율적인 Lock을 구현하기 앞서 어떤 Lock이 효율적인지 알아보겠습니다. 첫 번째는 기본 기능인 `Mutual exclusion`입니다. 올바른 구현으로 `critical section`에 단 하나의 쓰레드만이 접근 가능해야합니다. 두 번째는 `Fairness`입니다. 각 쓰레드가 기다리는 시간이 동일해야합니다. 마지막으로는 `Performance`입니다. 

## Controlling Interrupt

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/518a90f7-0303-4492-bd7b-fa44739b3bba">

초기 구현 방법은 인터럽트를 비활성화 하는 것입니다. `critical section`에 진입하기 전  **하드웨어 명령어**를 사용해 인터럽트를 비활성화하면 `critical section` 내부에서는 중단되지 않고 원자적으로 실행됩니다. 실행을 완료하고 `unlock()`을 통해 인터럽트를 다시 활성화하고 나서야 다른 쓰레드가 진입해 실행할 수 있습니다.

구현이 단순하지만 단점이 많습니다. 첫 번째는 `privileged operation`입니다. 예전 글에서 CPU의 가상화를 위해 시분할 방식을 채택했고 `Performance`와 `Control`을 고려해 `user mode`와 `kernel mode`로 분리한 `Limited Direct Execution`을 살펴 본 적이 있습니다. 권한을 분리했지만 다시 `user program`에서 `privileged operation`을 사용하는 문제가 발생한 것입니다. 결국 임의의 프로그램을 신뢰해야만 하고 이 기능을 남용할 시 막을 방법이 없게 됩니다.

두 번째는 멀티프로세서에서는 동작하지 않는 문제입니다. CPU1의 인터럽트를 비활성화 하더라도 다른 쓰레드가 CPU2에서 실행된다면 결국 `critical section`에 여러 쓰레드가 진입할 수 있습니다. 요즘은 멀티프로세서가 보편화 되어있으므로 결국 다른 구현 방법을 찾아야합니다.

세 번째는 인터럽트 손실입니다. 단순히 다른 쓰레드의 진입을 막기 위해 인터럽트를 비활성화 한다면 다른 기능들 또한 사용하지 못합니다. 다른 문제로 인터럽트가 발생하더라도 이를 감지 못하면 결국 수 많은 오류로 이어집니다.

마지막은 성능입니다. 일반적인 명령어에 비해 인터럽트를 마스킹하거나 마스킹을 해제하는 코드는 최신 CPU에서 느리게 실행됩니다. 위 이유로 인터럽트를 비활성화 하는 것은 운영체제에서 자체 데이터 구조에 엑세스할 때 원자성을 보장하는 등 제한된 상황에서만 사용됩니다. 

## A Failed Attempt, Just Using Loads / Stores

인터럽트 기반 Lock 대신 단순한 Load와 Store로 Lock을 구현해 봅시다. 첫 번째로 생각할 수 있는 접근은 단일 플래그 변수를 사용하는 방법입니다. 

<img width="506" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ac1c7733-a63c-4ca7-9f72-822843f7649c">

플래그를 통해 Lock을 특정 쓰레드가 소유중인지 여부를 판단할 수 있습니다. 처음 접근하는 쓰레드는 플래그 값을 확인해 0이라면 플래그를 1로 설정한 후 `critical section`에 진입합니다. 이후 다른 쓰레드가 `lock()`을 호출하면 플래그 값이 1이므로 CPU를 점유한 채 `spin-wait`를 합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/33afeb66-a1fe-45ba-bb9d-b4b1052d346e">

`spin-wait`를 해서 성능 문제가 있는 건 차치하더라도 기본적인 상호 배제도 성립하지 않습니다. 첫 번째 쓰레드가 플래그를 0으로 판단하고(TEST) 다른 쓰레드의 접근을 막기 위해 플래그를 1로 설정(SET)하는 사이에 인터럽트가 발생하면 다른 쓰레드 또한 `critical section`에 접근하게 됩니다. 즉, `TEST AND SET`이 원자적인 연산이 아니므로 이러한 문제가 발생합니다.

## Building Working Spin Lock with Test-And-Set

<img width="558" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/86107521-ca8b-4cd1-a120-a67d6ba5ec1f">

하드웨어의 도움이 필요한 시점입니다. Test와 Set을 원자적으로 할 수 있는 `TestAndSet` 명령어를 통해 상호 배제가 가능한 Lock을 구현해봅시다. 이해하기 쉽게 코드로 표현하면 명령어는 위 그림과 같습니다. 메모리에 존재하는 값을 리턴하고 새로운 값을 메모리에 저장하는 명령어임을 알 수 있습니다.

<img width="462" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5e223f5e-469a-4f91-b772-2246786ad875">

`A Failed Attempt`에서는 TEST와 SET 사이에 다른 쓰레드가 TEST + SET을 했지만 하드웨어가 제공하는 `TestAndSet`  명령어는 원자적이므로 이러한 상황이 발생하지 않습니다. 만약 플래그를 `0→1` 로 바꾼다면 반환값이 0이므로 `while`을 벗어나 `critical section`에 진입하고 `TestAndSet` 결과가 `1→1` 이라면 반환 값 역시 1이므로  `while`에 갇혀 `spin-wait`를 합니다.

## Evaluating Spin Locks

`TestAndSet` 기반의 Spin Lock은 `critical section`에 하나의 쓰레드만 진입하므로 상호 배제 기능을 제공합니다. 하지만 대기 중인 쓰레드마다 기다리는 시간이 일정하지 않아 `Fariness` 면에선 실패했다고 볼 수 있습니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e27571e5-e74e-4aeb-b625-776d96725270">

위 그림처럼 쓰레드 A가 `time slice`안에 `unlock()`이후 `lock()`을 호출하게 되면 B는 오래 기다렸지만 스케줄링 되지 않는 starvation에 빠지게 됩니다.

성능은 좀 더 생각해볼 거리가 있습니다. 만약 단일 프로세서에서 첫 번째 쓰레드가 Lock을 얻는다면 두 번째 쓰레드는 `spin-wait`를 해야하므로 성능이 좋지 않습니다. 하지만 멀티프로세서에서는 성능이 좋을 수도 있습니다. CPU1의 쓰레드 A와 CPU2의 쓰레드 B가 Lock을 놓고 경쟁하는 상황을 가정해봅시다. 쓰레드 A가 Lock을 획득한 후 쓰레드 B가 Lock을 얻으려고 하면 CPU2에서 `spin-wait`를 합니다. 그러나 `critical section`의 길이가 매우 짧다면 CPU2를 다른 쓰레드에게 양보하지 않더라도 잠시 대기했다가 Lock이 해제되는 순간 사용하면 되므로 효과적일 수도 있습니다. 즉 `context switching` 비용과 `critical section`의 길이에 따라 성능이 달라질 수 있습니다.

## Compare-And-Swap

<img width="538" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3040dbb5-6160-4230-98ac-5443d80854d8">

CAS 명령어는 메모리에 존재하는 값과 `expected` 값이 일치하면 `new`로 업데이트 하는 명령어입니다. `TestAndSet`와 유사하지만 보다 발전된 방식으로 다른 동기화 도구를 구축하는데 사용할 뿐 아니라 `lock-free algorithm`을 구축할때도 사용됩니다. `lock-free algorithm`은 낙관적인 접근 방식입니다. 일단 변수를 업데이트를 한 다음 충돌 감지를 사용해 다른 쓰레드가 동시에 변수를 업데이트 하는지 확인합니다. 충돌 없이 성공적으로 업데이트될 때까지 작업을 반복해서 다시 시도합니다. 따라서 시스템 전체적으로 항상 적어도 하나의 쓰레드가 작업하는 것을 보장합니다. 하지만 쓰레드가 많아진다면 `starvation` 현상을 피할 수 없습니다. 

<img width="464" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/d6115174-a5a2-45d7-be82-a7f608755dbc">

더 발전된 방법은 `wait-free algorithm`입니다.  `lock-free algorithm`에서 발생하는 `starvation`현상을 해결하여 각 쓰레드는 유한한 단계 내에 작업을 완료하도록 합니다. 하지만 현실적인 구현은 어려워 매우 한정적인 상황에서만 사용합니다.

이와 반대로 다른 쓰레드가 변수를 동시에 업데이트하고 있다고 가정하여 업데이트하기 전에 비관적으로 잠금을 거는 `Mutual exclusion locking`도 있습니다. 

Java의 `AtomicInteger`도 CAS를 사용한 대표적인 `lock-free` 기반 클래스입니다.  마지막으로 `CAS`와 유사하게 읽어온 메모리에 대해서 값이 변화가 없다면 업데이트를 하는 `Load Linked / Store Contidional`기계어도 알아두면 좋을 것 같습니다.

## Fetch-And-Add

<img width="280" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ac23e698-f309-44bb-803c-08c628ee77dd">

이 명령어는 값을 원자적으로 증가시키면서 이전 메모리의 값을 반환합니다. 이 명령어를 활용해 각 쓰레드의 실행을 보장하는 `ticket-lock`을 살펴보겠습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/a43802a7-28f9-43f5-bb5a-95479e2a8c5d">

`ticket-lock`은 두 개의 변수를 조합하여 잠금을 구축합니다. 쓰레드가 Lock을 획득하고자 할 때 번호표를 발급해주고(1) 전역적으로 공유되는 `lock→turn`을 확인하며 자신의 차례가 올 때까지 `spin-wait`를 합니다 (2). 따라서 쓰레드에 티켓이 할당된다면 미래의 어느 시점에는 스케줄링이 보장됩니다.  

## A Simple Approach: Just Yield, Baby

하드웨어로만 Lock을 구현해도 제대로 작동하는 것을 확인할 수 있었습니다. 하지만 `spin-wait`를 하므로 성능상 문제가 발생합니다. 경쟁중인 쓰레드가 `N`개인 상황에서 하나의 쓰레드가 `Lock`을 획득했다면 `N-1`개의 쓰레드는 `time slice` 동안 `spin-wait`를 해야만 합니다. 따라서 **OS의 도움**이 필요한 시점입니다!

<img width="414" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/89b88740-33f1-4204-b972-1f246b416388">

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3907f092-c92a-4752-b9cc-467935caefed">

첫 번째  방법은 자신이 Lock을 획득하지 못했다면 OS의 도움을 받아 즉시 `yield()`를 호출하는 방법입니다. `yield()`를 호출한 쓰레드는 `RUNNING`에서 `READY` 상태가 되어 deschedule됩니다.  N-1개의 쓰레드가 `spin-wait`를 하는 방식보다 낫지만 N-1개의 쓰레드의 컨텍스트 스위칭 비용 또한 무시할 수 없습니다. 뿐만 아니라 공평하지 못해 여전히 `starvation`문제가 발생할 수 있습니다.

## Using Queue, Sleeping Instead Of Spinning

`spin-wait` 또는 `yield()` 방식의 문제점은 스케줄러를 제어할 수 없어 우연에 기대는 점입니다. 만약 스케줄러가 잘못된 선택을 한다면 영원히 `starvation`문제를 극복할 수 없기 때문입니다. 

따라서 현재 Lock을 소유한 쓰레드가 Lock을 해제하면 다음 쓰레드가 Lock을 명시적으로 획득할 수 있도록 제어해야합니다. Lock을 획득하려고 시도했던 쓰레드가 대기할 수 있는 `waiting-queue`, 쓰레드를 절전모드로 전환할 수 있는 `park()`, 절전모드에 있는 쓰레드를 깨울 수 있는 `unpark()` 를 통해 더 효율적인 구현 방법인 `Mutex`를 살펴보겠습니다.

<img width="296" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fb4c93d0-3d70-4868-8d39-c9c2692923ee">

Lock의 사용 유무를 나타내는 `flag`와 획득을 시도한 쓰레드들이 대기하는 큐를 확인할 수 있습니다. `guard`는 `lock()` 과 `unlock()`을 원자적으로 연산할 수 있게 해주는 보조 Lock입니다. 뒤에서 다시 살펴보겠습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/df1a7591-81cc-4495-9a9a-4994cab8c985">

`lock()`은 Lock을 획득할 수 있다면 flag를 1로 바꿔 `critical section`에 진입하고 만약 Lock이 사용중이라면 현재 쓰레드를 대기열에 추가하고 `park()`를 통해 절전모드에 진입합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/bafc53f7-5e91-450f-9ec7-4dd0daed137c">

공유 변수인 `flag`를 다룰 때 경쟁 상태가 발생할 수 있습니다. 올바르게 값을 업데이트 하기 위해 `guard`를 추가적으로 사용하며 공유 변수 업데이트를 `Atomic`하게 보장합니다. 

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/2de0f3f3-9d4e-4009-a7f7-336149f294e5">

`Mutex`도 `spin-wait`를 하지만 괜찮은 이유는 무엇일까요? 위 그림에서 빨간색 부분은 `lock()`과 `unlock()` 사이에 존재하는 `critical section`을 나타냅니다. 만약 `lock()`이 `waiting-queue`를 사용하지 않고 `spin-wait`를 한다면 멀티 프로세서 기준에서 `critical section`만큼 기다린 후에 `unlock()`이 됩니다. 최악의 상황에서는 언제 `unlock()`이 되어 다음 쓰레드가 진행될지 알 수 없지만 `guard`를 활용한다면 짧은 시간 동안만 `spin-wait`를 하는 것이 보장되므로 나쁘지 않은 선택입니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/693811ef-2d38-487d-8dbd-3c2613917960">

이 코드는 여전히 문제가 있습니다. 만약 Lock이 사용중이여서 첫 번째 쓰레드는 `waiting-queue`에 들어가기 위해 준비중이라 합시다. `park()`를 호출하기 직전 `context-switching`가 발생한 후 두 번째 쓰레드가 `unlock()`을 호출하면 어떻게 될까요? `queue`는 비어있지 않으므로 `else(2번)`가 실행됩니다. id를 제거하고 대기중인 쓰레드를 깨우기 위해 `unpark()`를 호출합니다. 이후 다시 첫 번째 쓰레드로 전환되면 어떠한 쓰레드도 깨울 수 없지만 `park()`를 호출하게 됩니다.

<img width="410" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c427d8c4-5a06-4cea-aa49-7d8e74515b7b">

Solaris 운영체제는 `setpark()` 시스템 호출로 이 문제를 해결합니다. `setpark()`와 `park()` 사이에 `unpark()`가 호출되지 않았다면 `park()`는 예정대로 실행되지만 그 사이 `unpark()`가 호출됐다면 `park()`는 아무런 역할을 하지 않고 리턴합니다.

