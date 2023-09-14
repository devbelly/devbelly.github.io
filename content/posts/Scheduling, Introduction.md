---
title: Scheduling, Introduction
date: 2023-09-10T21:37:37+09:00
draft: false
comments: true
toc: true
tags:
  - os
---

안녕하세요. 이번시간에는 스케줄링에 대해 알아보겠습니다. 컨텍스트 스위칭을 설명하면서 CPU가 어떤 프로세스에게 할당될지 OS의 스케줄러에 의해 결정된다고 간단히 언급했습니다. 스케줄러가 사용하는 정책에는 어떤 것들이 있는지와 이러한 정책들을 평가하는 메트릭에 대해 알아보겠습니다.

정책들을 살펴보기에 앞서 각 프로세스(이번 글에서는 job이라 부르겠습니다)는 아래 상황을 가정하며 시작하도록 하겠습니다. 

1. 모든 job들은 실행 시간이 동일하다.
2. 모든 job들은 도착 시간이 동일하다.
3. 한번 job이 실행되면 끝날때까지 실행된다.
4. 모든 job들은 CPU집약적인 연산을 한다.(I/O 연산을 하지 않는다고 가정)
5. 모든 job들의 소요 시간을 알 수 있다.

겉보기에도 비현실적인 이 가정들을 나중에 하나씩 없애보며 스케줄러가 사용하는 현실적인 정책들에 대해 살펴보겠습니다.
## Metrics

정책을 본격적으로 살펴보기 전 해당 정책들을 평가하려면 기준이 있어야합니다. 메트릭은 크게 Performance와 Fairness 관점에서 살펴볼 수 있는데 두 메트릭은 Trade off 관계에 놓여있는 경우가 많습니다. Performance을 올리면 Fairness가 감소하지요. 일단 Performance와 관련한 메트릭인  `turnaround time`에 대해 살펴보겠습니다.

<img width="542" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/b93a7046-0692-45fe-9a12-171089945f0e">

`turnaround time`은 job이 끝난 시간에서 job이 도착한 시간을 뺀 값입니다. 즉 어떤 job이 완료될 때까지 걸린 시간을 의미합니다.
## First In, First Out(FIFO)

메트릭에 대해서도 간단히 알아봤으니 이제 정책이 좋고 나쁨을 따질 수 있습니다. 첫 번째로 살펴볼 정책은 FIFO입니다. 직관적이여서 구현하기 쉽다는 장점이 있고 처음에 언급한 5개 가정하에선 FIFO도 나쁘지 않은 정책입니다. 

<img width="404" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/e301f83e-e99e-4b46-938b-1d14316e732f">

A, B, C job이 동시에 도착했다고 가정해봅시다. 위 상황에서 turnaround time을 계산해볼까요? A의 반환 시간은 10, B의 반환 시간은 20, C의 반환 시간은 30이므로 평균 반환 시간은 20이 됩니다. 괜찮은 수치인건 맞지만 가정한 상황 자체가 너무 이상적입니다. FIFO의 단점을 확인해보기 위해 첫 번째 가정이였던 `모든 job의 실행 시간이 동일하다`를 없애 각 job들의 실행 시간이 다른 경우를 생각해보겠습니다.

<img width="386" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/8b05ced1-b2ad-4563-970f-7fbb7a9f54f9">

A의 실행 시간이 100으로 늘어났습니다. 각 job들의 반환 시간을 계산해보면 100, 110, 120으로 평균 반환 시간이 110으로 크게 늘었습니다. B와 C는 앞에서 먼저 스케줄링된 A 때문에 오래 기다려야하는 상황이여서 평균 반환 시간이 늘어났습니다. 이를 `convey effect`라고 합니다.

## Shortest Job First(SJF)

위 상황에서 평균 반환 시간을 줄이려면 어떻게 해야할까요? 시간이 짧게 걸리는 job부터 먼저 처리를 해주면 됩니다. 이를 Shortest Job First 정책이라 합니다.

<img width="386" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/adc320b2-ae01-41ea-8bf5-417de284e250">

시간이 짧게 걸리는 B, C를 먼저 스케줄링하고 오래걸리는 A를 가장 나중에 실행한 모습입니다. 평균 반환시간을 계산해보면 A는 120, B는 10, C는 20이므로 50이 됩니다. FIFO의 worst case보다 반환 시간이 크게 감소한 모습을 알 수 있습니다.

SJF는 도착 시간이 같다는 전제하에서는 최적의 반환 시간을 보장합니다. 하지만 여전히 앞선 가정들이 비현실적입니다. 2번째 가정이였던 도착시간이 동일하다는 조건을 없애 모든 job이 언제든 도착할 수 있다고 해봅시다. 만약 도착시간이 A가 B, C보다 빨랐다면 여전히 FIFO에서 발생한 문제들이 나타납니다.

<img width="374" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/4346db4d-d31d-4849-84bf-50e45e37a791">

B와 C가 10에 도착했다면 각 job의 반환 시간은 A는 100, B는 100, C는 110이므로 평균 반환 시간은 103.33이 됩니다. 

## Shortest Time-to-Completion First(STCF)

위 상황을 개선하려면 어떻게 해야할까요? 반환 시간을 개선하기 위해서는 실행 시간이 짧은 것을 먼저 실행시키는 것이 좋지만 세 번째 가정이였던 `한 번 job이 실행되면 끝날때까지 실행된다`라는 조건 때문에 계속해서 A를 실행했습니다. 이러한 job을  `non-preemptive` 하다 라고 표현합니다.

SJF는 `non-preemptive` 스케줄러입니다. 이를 `preemptive` 버전으로 변경한 것이 `Short Time to Completion First`입니다. 현재 가지고 있는 job중 남은 시간이 가장 짧은 job부터 실행하는 스케줄러로 위 예제에서는 A가 실행중이더라도 B, C가 도착한 순간부터 남아있는 시간이 가장 짧은 job이 B와 C이므로 A를 중단하고 B, C를 먼저 실행합니다. 

<img width="384" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c9c8cc82-b0a6-4031-866c-a9ff5380ef8b">

## Response Time

지금까지 모든 job이 CPU집약적인 연산을 수행하고 job의 소요 시간을 안다면 반환 시간 기준으로 STCF가 최적임을 살펴봤습니다. 하지만 만약 실행시간이 긴 job이 도착한 상황에서 계속 짧은 job들이 실행된다면 실행시간이 긴 job은 스케줄링되지 않는 `starvation` 현상이 일어납니다.

<img width="546" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c344c5c2-f409-4e1d-a9fc-bdce18f6b5ea">

이에 Performance의 두 번째  메트릭인 Response time이 등장했습니다. 반응 시간은 도착 시간부터 첫 번째 실행까지 걸린 시간을 의미합니다. 
## Round Robin

반응 시간을 기준으로 좋게 평가 받는 스케줄러가 Round Robin입니다. timer interrupt를 통해 시분할 시스템을 구현하며 프로세스들 사이에 우선순위를 두지 않고 일정 시간단위로 CPU를 할당하는 방식의 스케줄러입니다.

<img width="426" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/39ee4e01-378c-4f49-982e-305c5c1dc4a8">

다시 처음 예제로 돌아와 A, B, C job이 동시에 도착했다고 가정해봅시다. SJF는 반환 시간은 좋지만 반응 시간은 느립니다. 만약 우리가 C를 요청했는데 SJF에서는 앞에 두 job이 실행될 때 까지 기다린 후 C가 실행된다면 반응 시간 측면과 interactive 면에서 좋지 않을 것입니다. 반응 시간을 계산해보면 각각 0, 5, 10 이므로 평균 반응 시간은 5 입니다.

`Figure 7.7`에서는 이를 개선한 Round Robin을 사용했습니다. 각 job의 반응 시간은 0, 1, 2로 평균 반응 시간이 1로 크게 개선이 된 모습입니다. 다만 위 그림에서 표시하지 않은 부분이 있는데 이전 글에서 살펴본 Context Switching 비용입니다. RR에서 context switching을 빨리 할수록 반응 시간이 빨리지지만 레지스터를 바꾸는 컨텍스트 스위칭 비용과 메모리에 현재 실행중인 프로세스를 위한 것들 또한 바뀌어야하는 오버헤드가 발생합니다. 즉, 적절하게 설정해주는 것이 중요합니다.

<img width="546" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/8735c5ab-8623-48f3-88b1-202c9f4d5895">

## Incorporating I/O

이제 네번째 가정를 없애 I/O 작업도 job에서 실행해볼까요? 현실 세계에서의 프로그램은 입출력을 사용하기 때문입니다.

I/O도 여러 종류가 있지만 기본적으로 DMA를 통한 I/O는 CPU에 의존적이지 않습니다. 참고로 DMA는 Direct Memory Access의 약자로 CPU의 개입없이 I/O 장치와 기억장치 사이에 데이터를 전송하기 위한 목적으로 만들어진 장치입니다. (아에 개입이 없는 것은 아니지만 거의 의존하지 않습니다.)

<img width="370" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/37782d2f-527f-4620-8c79-e42e82724665">

A와 B 둘다 50ms을 사용하는 job을 상황입니다. A는 10ms의 job이 5개 존재하고 B는 50ms 하나의 job으로 이루어져있습니다. STCF 스케줄러에 A job과 B job이 도착했을 때 10ms의 작은 A와 50ms의 큰 B가 있으므로 스케줄러는 A를 먼저 실행합니다. 실행 도중 A는 I/O 요청을 하는데 이때 A는 CPU를 사용하지 않고 blocked 상태로 바뀝니다. 

이후 스케줄러에는 B만 존재하므로 B가 스케줄링 되어 실행됩니다. 10ms이 지난 후 I/O 요청이 완료되었다는 interrupt가 발생하면 A는 blocked에서 ready 상태가 되어 다시 스케줄링될 준비가 되었습니다. 다시 A의 Completion time이 짧기 때문에 A가 스케줄링되어 실행됩니다.

이렇게 되면 I/O 요청시간에 CPU를 활용함으로써 전체적인 반환 시간을 낮출 수 있게 됩니다. 

## No more oracle

이제 마지막 가정인 `모든 job들의 소요 시간을 알 수 있다.`만 남겨둔 상태입니다. 이 가정이 가장 해결하기 어려운 가정입니다. 다음 시간에 살펴볼 `Multi-Level-Feedback-Queue`에서 이 가정을 어떻게 해결하는지 살펴보도록 하겠습니다.

## 참고

DMA
- https://www.youtube.com/watch?v=LqsDmPFKmXU
