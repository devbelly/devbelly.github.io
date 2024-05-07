---
title: Concurrency, An Introduction
date: 2024-05-01T11:28:57+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
이번 시간에는 단일 프로세스 내에서 여러 개의 실행 흐름을 갖는 쓰레드에 대해 알아보겠습니다. 

쓰레드는 프로세스와 매우 유사합니다. 하나의 프로세스에서 다른 프로세스를 실행해야할 때, 필요한 정보들을 PCB에 저장하고 다른 프로세스의 PCB를 가져와야합니다. 쓰레드 또한 계산에 사용하는 고유 레지스터 세트가 있고 단일 프로세서에서 실행중인 두 개의 쓰레드가 있는 경우, 쓰레드간 컨텍스트 스위치가 일어나야 합니다. 따라서 현재 쓰레드의(T1) 레지스터 상태를 TCB에 저장하고 다른 쓰레드(T2)의 레지스터 상태를 TCB로부터 복원해야합니다. 하지만 쓰레드들은 동일한 주소 공간을 사용하므로 PCB처럼 주소변환정보를 TCB에 저장할 필요는 없습니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/7350037f-e288-4885-963d-692f8ebf98ce">

또 다른 차이점은 스택입니다. 쓰레드는 하나의 실행 흐름이라 볼 수 있습니다. 프로그램이 실행된다는 것은 스택에 흔적을 남기는 행위입니다. 실행에 필요한 파라미터를 전달받고 스택에서 계산을 하며 반환 값과 반환될 주소 또한 스택에 저장합니다. 따라서 멀티쓰레드 프로세스는 쓰레드마다 독립적인 스택을 가져야합니다.  

## Why Use Thread?

쓰레드를 사용하는 이유는 크게 두 가지가 있습니다. 첫 번째는 병렬 처리입니다. 예를 들어 매우 큰 배열에 연산을 수행하는 프로그램을 작성한다고 가정하겠습니다. 단일 프로세서에서는 단순하게 처리하면 되지만 멀티 프로세서 환경에서는 해야하는 작업의 일부를 각각 실행한다면 성능 향상을 기대할 수 있습니다. 표준 단일 쓰레드 프로그램을 여러 CPU에서 작업을 수행하는 프로그램으로 변경하는 작업을 `병렬화`라고 하며, 최신 하드웨어에서 병렬화를 통해 프로그램 수행 속도를 올릴 수 있습니다.

여기서 한 가지 드는 생각은 CPU 갯수 이상 쓰레드를 만들 필요성이 있는가 입니다. 이상적으로 I/O 및 동기화 작업이 없는 환경에서는 코어당 하나의 쓰레드를 사용하는 것이 이상적입니다. 하지만 일반적으로 I/O 및 동기화 작업이 이뤄지는 경우가 많아 코어 갯수보다 많은 쓰레드를 사용합니다.([참고](https://stackoverflow.com/questions/1718465/optimal-number-of-threads-per-core))


두 번째 이유는 느린 I/O로 인해 프로그램 진행이 차단되는 것을 피하기 위해서입니다. I/O 요청을 보낸 후 완료될 때까지 기다리는 동안 CPU 스케줄러가 실행 준비된 다른 쓰레드를 스케줄링하면 프로그램이 중단되는 것을 방지할 수 있습니다. 

## Why It Gets Worse, Shared Data

여러 쓰레드가 공유 변수에 접근하여 증가하는 예시 코드를 살펴보겠습니다.

<img width="528" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/992e76f4-43bb-4eb7-92d8-9fe1e04abc75">

위 코드를 실행하면 두 쓰레드가 협력하여 동일한 공유 변수를 증가해 20,000,000이 될 것이라 기대할 수 있습니다. 결과가 어떤지 살펴볼까요?

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c05cf0ed-ef0f-4054-89ac-4b5930ce60a2">

결과가 20,000,000에 못미칠 뿐 아니라 실행할 때마다 값이 변합니다. 왜 이런 비결정적인 결과가 발생하는 걸까요?

## The Heart Of The Problem, Uncontrolled Scheduling

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/6d803c90-f830-4106-b09e-5b5fce2417c1">

상위 수준에서 변수를 1 증가시키는 행동은 한 번에 일어나는 것처럼 보이지만 컴파일러 수준에서는 위 그림과 같이 3개의 동작이 순차적으로 일어납니다. 따라서 위 그림과 같은 코드 시퀀스가 원자적으로 진행되어야 의도한대로 동작합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/0ecab8cc-9c43-4920-8f73-1f3650a73cc9">

CPU 스케줄러는 상황에 따라 실행하는 프로세스나 쓰레드의 순서를 달리합니다. 위 그림은 두 개의 쓰레드가 하나의 공유 변수를 증가시키는 시나리오에서 첫 번째  쓰레드가 50을 51로 증가시키기 위해 코드 시퀀스를 실행 하던 도중 마지막 `mov` 명령을 실행하지 못한 상태에서 timer interrupt가 발생한 상황입니다. 따라서 OS는 현재 실행중인 쓰레드의 상태(PC, eax를 포함한 레지스터)를 $TCB_1$에 저장합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5c0c499e-90ef-4469-8e43-58cfcdb6b93b">

운이 좋게도 Thread 2는 코드 시퀀스 실행을 방해받지 않고 원자적으로 실행했습니다. 따라서 다시 Thread 1이 실행 기회를 얻어 $TCB_1$에 저장된 레지스터 값을 복원합니다. 복원된 값은 Thread 2 실행 시작 전 PC와 `eax`레지스터 값과 일치합니다. 마지막으로 남은 `mov` 명령을 실행했지만 의도한 52와는 달리 최종 결과는 51임을 알 수 있습니다.

이처럼 여러 프로세스나 쓰레드가 공유 자원에 동시 접근할 때 실행 순서에 따라 결과가 달라지는 현상을 `race condition`이라 합니다. `race condition`을 피하려면 공유 리소스가 존재하는 `critical section`에 여러 쓰레드가 동시에 접근해선 안됩니다. 여러 쓰레드가 동시에 `critical section`에 접근하는 것을 보장하는 속성을 `mutual exclustion`이라 합니다.

## The Wish For Atomicity

이를 해결하기 위한 방법 중 한가지는 하드웨어 수준에서 `atomic operation`을 지원하는 것입니다. 예를 들어 메모리에 있는 값을 1 증가시키기 위해 아래와 같은 명령어를 제공할 수 있습니다.

<img width="288" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/ac257c7d-1de0-4006-a5f0-3e2f2b26e716">

하드웨어는 명령어가 원자적으로 실행됨을 보장합니다. 따라서 실패와 성공만 존재하며 그 중간이 존재하지 않습니다. 좋은 해결책이라 생각할 수 있겠지만 범용성이 떨어집니다. 만약 `concurrent B-tree`를 원자적으로 업데이트할 필요성이 있다면 instruction set 또한 `update for concurrent B-tree` 명령어를 지원하는 방향이 맞을까요? 그렇지는 않습니다.

따라서 현실적인 해결책은 하드웨어의 `few useful instruction`와 더불어 `operating system`의 도움을 받아 `critical section`을 관리하는 방법입니다. 이제부터 천천히 이 방법에 대해 살펴보겠습니다

## One More Problem, Waiting For Another

쓰레드간 상호 작용에는 공유 데이터를 다루는 문제도 있지만 순차 실행도 있습니다. 예를 들어 하나의 쓰레드가 실행되기 위해서는 다른 쓰레드의 작업이 완료돼야 하는 경우도 있습니다. 앞으로 `synchronization` 문제 뿐 아니라 `sleep/wake` interaction에 대해서도 알아보겠습니다.