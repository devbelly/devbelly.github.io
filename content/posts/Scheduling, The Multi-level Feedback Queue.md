---
title: Scheduling, The Multi-level Feedback Queue
date: 2023-09-12T14:09:07+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
안녕하세요. 이번시간에는 `Multi-level Feedback Queue(MLFQ)`에 대해 알아보겠습니다. MLFQ의 목표는 두 마리의 토끼인 반환 시간과 반응 시간을 둘 다 줄이는 것입니다. 이전 글에서 살펴보았듯이 반환 시간을 줄이기 위해서는 실행 시간이 짧은 job에게 먼저 CPU를 할당해야하는데 OS는 어떤 job이 짧은지 미래를 예측할 수 있는 능력을 갖고 있진 못합니다. 또한 interactive job을 요청한 사용자를 위해 반응 시간을 줄이고 싶지만 반응 시간을 줄이면 줄일 수록 반환 시간이 늘어나는 문제에 대해 살펴봤습니다.

## Multi-Level Feedback Queue

이러한 문제들을 해결하기 위해 MLFQ가 고안됐습니다. 여러 개의 큐를 사용하며 각 큐마다 우선순위가 달라 높은 우선순위 큐에 job이 있다면 해당 job 부터 실행하도록 합니다. 만약 특정 대기열에 여러 개의 job이 있다면 Round Robin을 통해 순차적으로 실행하도록 합니다. 즉 요약하자면 아래 두개의 rule로 요약됩니다

- 규칙 1: A가 B보다 우선순위가 높은 큐에 있으면 A가 실행됩니다(B는 실행되지 않음).
- 규칙 2: A와 B가 동일한 큐에 있다면, A와 B는 RR으로 실행됩니다.

<img width="334" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/acd5a03c-a022-4f48-b6bb-524a6287cf68">

MLFQ의 핵심은 미래 예지 없이 우선순위를 부여하는 방식입니다. 각 job에 고정된 우선순위를 부여하는 것이 아니라 최근까지의 행동을 관찰하여 우선순위를 결정합니다.  만약 어떠한 job이 interactive해서 CPU를 반복적으로 포기한다면 MLFQ에서 높은 우선순위에 두지만 timer interrupt에 의해 CPU를 반납했다면 Long job으로 판단하여 낮은 큐로 강등 시킵니다. 요약하자면 아래 rule로 정리할 수 있습니다.

- 규칙 3: 스케줄러에 job이 도착하면 가장 높은 큐에 위치시킨다.
- 규칙 4a: 만약 job이 `time-slice`를 전부 사용하면 낮은 큐로 이동시킨다.
- 규칙 4b: 만약 `time-slice`가 끝나기 전 CPU를 반납하면 같은 큐에 위치시킨다.

위 5가지 규칙을 가진 MLFQ를 통해 여러 job들이 어떻게 스케줄링되는지 확인해보겠습니다.

## Example 1. A Single Long-Running job

첫 번째 예시는 하나의 긴 단일 job입니다. 

<img width="348" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fe0a4760-9db1-4c37-94ac-4cc1da12811f">

`rule 3`에 의해 처음에 들어온 job은 가장 높은 우선순위인 Q2에 위치합니다. 이후 time-slice로 정해진 10ms만큼 사용하게 되면 Long Job으로 판단되어 Q1, Q0로 강등된 모습을 알 수 있습니다. 

## Example 2. Along Came A Short Job

이제 A가 실행되는 도중 interactive job B가 들어온 상황을 살펴보겠습니다. 

<img width="338" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3008e2cb-f5c1-4243-a355-1fe7db09bd93">

상대적으로 CPU를 많이 사용하는 A보다 짧은 시간 실행되는 B가 100ms에 도착한 모습입니다. 정해진 time-slice인 10ms을 사용한 후 Q1 까지는 강등되어 실행되지만 실행되는 시간이 짧아 Q0까지 강등되기전 실행이 종료된 모습입니다.

이 그림에서 MLFQ가 반환 시간을 어떻게 최적화 하려고 시도하는지 알 수 있습니다. 이전 글의 소제목 이였던 `No More Oracle`에서 살펴봤듯 OS는 실행되는 job의 길이가 얼마나 될지 알 수 없습니다. 하지만 SJF나 STCF 알고리즘은 실행되는 job의 길이에 의존하는 알고리즘이였죠. 

마찬가지로 MLFQ는 처음에 들어온 job이 얼마나 오래걸릴지 알 수 없습니다. 하지만 job이 빨리 끝난다면 상위 큐에서 실행되어 끝나게 되고 job이 길다면 점차 하위 큐로 내려가 머무르게 됩니다.이러한 시스템을 통해 MLFQ는 미래에 대한 예지는 없지만 최대한 SJF처럼 시간이 짧은 job에게 우선순위를 주고자 했습니다.

## Example 3. What about I/O?

이제 I/O가 발생한 상황에 대해 알아보겠습니다. interactive job B는 사용자로부터 입출력을 자주 주고 받습니다. 

<img width="274" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/14bd70a1-50ac-4b1a-8826-6e13d2a215f7">

`rule 4b`에 따라 job이 time-slice 이전에 CPU를 포기하면 동일한 우선순위에 두기로 했습니다. 만약 어떠한 이유에서 I/O 가 많은 job이 강등 당하면 interative 하지 않게 될 확률이 높기 때문입니다. 때문에 위 그림에서 job B가 I/O 이전에 1ms 만큼 CPU를 사용하고 반납한다면 강등 당하지 않고 Q2에 머물러 있습니다.

## Problems With Our Current MLFQ

지금까지는 두 개의 job만 실행되는 예시들을 살펴봤는데 만약 3개라면 어떻게 될까요? 

<img width="282" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/991ebad9-8d7e-4f1a-a7d8-cc7fe02eb841">

CPU를 많이 사용하는 job A와 interactive job B, C가 있다고 해봅시다. CPU를 많이 사용하는 A는 시간이 지남에 따라 Q0까지 강등이 된 상황에서 B와 C가 스케줄러에 도착했습니다. time-slice가 지나기 전에 CPU를 반납하므로 둘다 Q2에 머물러 있어 계속 B와 C만 실행되는 상황입니다. A는 실행되지 못하는데 이를 `starvation`이라고 합니다.

<img width="268" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/41bd9a51-a231-467f-90f8-71f3f3892ca8">

두 번째 문제는 악의적인 프로그래머가 일부러 CPU를 계속 반납하는 행동을 하는 것입니다. 이를 `Gaming the scheduler`라 합니다. `time-slice`가 끝나기 전에 CPU를 반납하면 현재 MLFQ는 short job이라 판단하여 우선순위를 강등시키지 않고 그대로 유지시킵니다. 실제로 `I/O`요청을 하면 별 이득이 없으므로 메모리에서만 처리가 되는 syscall을 사용하거나 `/dev/null`와 같은 가상 장치 파일을 이용합니다. 
## Solution 1.  The Priority Boost

우선 순위가 낮은 큐에 있는 job들이 `starvation`현상을 겪지 않게 하는 방법엔 무엇이 있을까요? 정답은 생각보다 간단합니다. 주기적인 Boost를 통해 굶지 않도록 하면 됩니다. 아래 새로운 규칙을 적용한 그림을 살펴보겠습니다.

- Rule 5: S만큼 시간이 지나면 모든 job들을 맨 위의 큐로 이동시킨다.

<img width="266" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/1ae73bab-030a-4106-adfe-8a409507c736">

Boost를 하게 되면 굶어 죽던 A가 다시 Q2로 이동하여 스케줄링 되므로 `starvation`현상을 해결할 수 있습니다. 뿐만 아니라 A가 CPU 집약적인 연산을 하다가 interactive job으로 변경되어도 Boost 이후 높은 큐에 위치하므로 적절한 반응 시간을 리턴할 수 있습니다.
## Solution 2. Better Accounting, time-allotment

나머지 문제인 `Gaming the scheduler`는 어떻게 해결하면 될까요? 원인부터 생각해보면 interactive job을 위해 CPU를 반납하면 그대로 우선순위를 유지하는 규칙이였던 `rule 4b`가 문제가 됩니다. CPU를 할당받을때마다 이전에 얼마나 사용했는지는 중요하지 않지요.

룰을 다음과 같이 수정해봅시다.

- 규칙 4: 작업이 주어진 수준에서 `time-allotment`를 모두 사용하면(CPU를 얼마나 많이 사용했는지에 관계없이) 우선 순위가 감소한다.(즉, 대기열에서 한 단계 아래로 이동합니다).

<img width="278" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/fd639110-074d-4486-b23f-db6d59b6677d">

흰색 job을 자세히 보면 얇은 선이 있습니다. 악의적인 유저가 일부러 CPU를 반납하여 강등을 안당하려고 하는 모습이지만 정해진 `time-allotment`를 소진하면 timer interrupt이든 CPU를 자진 반납하든 상관없이 우선순위가 낮아지는 모습을 확인할 수 있습니다. 이로써 Gaming에 대해 tolerance를 가지게 됩니다. 그리고 자세한 구현은 모르겠지만 아마 `I/O` 요청 이후 다시 CPU를 사용할 수 있는 상태가 되면 `I/O`를 요청했던 job이 우선적으로 CPU를 가져가는 모습을 볼 수 있습니다.

## 마무리

지금까지 MLFQ에 대해 살펴봤습니다. MLFQ는 가장 최근에 CPU를 어떻게 사용했는지 Feedback을 통해서 job들에게 priority를 부여함으로써 반환 시간과 반응 시간을 최적화하고자 했습니다. MLFQ마다 몇 개의 큐를 사용할 것인지, 강등당한 job을 얼마나 자주 boost할 것인지는 MLFQ마다 다릅니다. 또한 큐마다 `time-allotment`를 동일하게 하는 대신 낮은 큐는 CPU를 많이 사용할 것이라는 게 예상이 되므로 `time-allotment` 를 늘릴 수도 있습니다.

<img width="396" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/a6d129b1-5c04-4546-a43a-628e48c626ec">

