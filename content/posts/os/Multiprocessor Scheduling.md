---
title: Multiprocessor Scheduling
date: 2023-09-17T12:13:59+09:00
draft: false
comments: true
toc: true
tags:
  - os
---
안녕하세요. 이번 시간에는 Multiprocessor Scheduling에 대해 알아보겠습니다. 멀티프로세서는 두 개 이상의 CPU가 있는 컴퓨터 시스템을 가리키고 각각의 주변 장치 뿐만 아니라 메인 메모리도 공유합니다. 일반적인 애플리케이션은 단일 CPU만 사용하므로 멀티프로세서 환경일지라도 처리속도가 빨라지진 않습니다. 이를 개선하기 위해 작성된 프로그램을 멀티 스레드를 사용하도록 재작성해야합니다. 멀티 스레드 애플리케이션은 여러 CPU에 작업을 분산할 수 있기 때문입니다. 

## Background: Multiprocessor Architecture

위처럼 애플리케이션 레벨의 문제 말고도 OS수준에서도 문제가 발생합니다. 왜냐하면 지금까지 살펴본 스케줄링은 모두 단일 CPU를 가정했기 때문이죠. 

CPU는 1초에 여러 개의 instruction을 실행할 수 있습니다. 매 번 RAM에 접근해서 데이터를  읽어온 후 CPU에서 instruction을 수행하는 것은 비효율적이므로 성능 향상을 위해 다양한 전략을 사용합니다. 그 중 하나가 캐싱입니다. 캐시는 계층 구조를 띠며(L1, L2, L3) 메인 메모리에서 ***인기 있는*** 데이터의 사본을 저장하는 작고 빠른 메모리입니다. 반면 메인 메모리는 모든 데이터를 보유하지만 해당 데이터에 대한 엑세스는 느립니다. 때문에 캐시를 활용하여 느린 메모리를 빠른 메모리처럼 보이게 하는 효과를 얻게됩니다.

처음 프로그램이 실행되면 캐시에는 아무런 정보가 없고 원하는 데이터가 메인 메모리에 있으므로 데이터를 가져오는데 오래 걸립니다. 가져온 정보가 캐싱이 된 후 나중에 다시 동일한 데이터에 접근한다면 CPU는 먼저 캐시에서 해당 데이터를 찾게 되므로 훨씬 빠르게 접근할 수 있습니다. 

그렇다면 도대체 ***인기 있는*** 데이터가 무엇일까요? 이를 판단하기 위해 `locality`개념을 사용합니다. `locality`에는 `temporal locality`와 `spatial locality` 가 있습니다. `temporal locality`는 말 그대로 시간과 관련된 `locality`로 어떤 데이터에 접근했다면 가까운 미래에 해당 데이터에 다시 접근한다는 의미입니다. `spatial locality`는 공간과 관련된 `locality`로 어떤 데이터에 접근했다면 해당 데이터와 메모리 주소상 가까운 다른 데이터들도 접근한다는 의미입니다. 배열을 순회하는 프로그램이 이에 해당합니다.

<img width="486" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/0398e170-4fed-4310-bc95-0b28fbc9ea8e">

이제 CPU가 여럿인 상황에 대해 생각해볼까요? 각 CPU마다 고유의 캐시를 갖고 있습니다. 예를 들어 CPU 1에서 메모리 주소 A에 접근하고자 한다면 일단 캐시에 해당 데이터가 있는지 살펴봅니다. 해당 내용이 없으므로 A에 접근하여 D라는 값을 읽고 프로그램을 수행하여 D’로 업데이트를 합니다. 이 때 CPU 2에서 메모리 주소 A에 접근한다면 D’이 아니라 D를 읽어오는 문제가 발생합니다. 이러한 문제를 `cache coherence`라고 합니다.

이러한 `cache coherence`문제를 해결하기 위한 하드웨어 수준에서의 해결책은 `snooping`이 있습니다.  

## Don’t forget Synchronization

`cache coherence`를 문제를 해결하더라도 여전히 프로그램 수준에서 공유 데이터에 대한 동기화를 신경써야합니다.  자세한 내용은 책에서 동기화를 다룰때 다시 다시 살펴보겠습니다.

## Cache Affinity

멀티프로세서 스케줄링에서 마지막으로 고려해야할 점은 `cache affinity`입니다. 프로세스 A가 CPU 1에 스케줄링되면 해당 CPU의 캐시에는 프로세스 A와 관련된 정보들이 많이 있습니다. 다음에 프로세스를 실행할 때 필요한 정보가 이미 캐시에 존재한다면 더 빠르게 실행될 수 있으므로 동일한 CPU에 프로세스를 스케줄링 해주는 것이 효과적입니다. 

## Single Queue Scheduling

위에서 언급한 내용들을 바탕으로 멀티프로세서의 스케줄링에 대해 살펴봅시다. 첫 번째로 기존에 살펴봤던 단일 프로세서 스케줄링을 활용하는 방법입니다. 스케줄링이 필요한 작업들을 Single Queue에 넣는 방법으로 Single Queue Multiprocessor Scheduling(SQMS)라고 부릅니다. 장점은 단일 큐를 사용하므로 구현이 간단하다는 점입니다.

하지만 단점이 명확합니다. 첫 번째 단점은 `lack of scalability`입니다. 교재에서 `Don't forget Synchornization`부분을 살펴보면 두 개의 쓰레드에서 동일한 큐에 접근하는 예시를 보여줍니다. 마찬가지로 여러 CPU가 하나의 single queue에 접근하려면 동시성 문제가 발생하지 않도록 Lock을 사용해야합니다. CPU가 증가할수록 Lock에 대한 대기 시간이 길어지기 때문에 성능 관점에서 문제가 발생하게 됩니다.

두 번째 단점은 `cache affinity`문제입니다. 

<img width="410" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/3f14ce22-3aec-47c6-a3e4-fb0fcd3769ff">

Queue의 모습이 다음과 같다고 해봅시다. 발생할 수 있는 동시성 문제는 Lock을 통해 해결한 상태입니다.

<img width="408" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/b4e0917f-1dea-48b2-ada6-fad062f34da3">

CPU는 큐의 맨 앞에서 실행할 작업을 순차적으로 꺼내서 처리하게 됩니다. 멀티프로세서에서는 캐시 효율성을 고려해서 자신이 실행이 되었던 CPU에 다시 스케줄링이 되면 좋다고 말했었는데 SQMS에서는 이러한 `cache affinity`를 고려하지 않고 job들이 CPU를 돌아다니게 됩니다. 위 사진에서 어떠한 job이 특정 CPU에서 실행되는 모습을 볼 수 없습니다.

<img width="396" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c42cd7a3-e039-4274-b5fd-4dbdc60c3bd7">

이 문제를 해결하기 위한 방법은 당연히 특정 프로세스를 특정 CPU에서 실행해 `cache affinity`를 달성하는 것입니다. A부터 D까지는 `cache affinity`가 개선되어 캐시 성능을 기대할 수도 있고 경우에 따라서는 E가 어느정도 CPU를 돌아다녔다면 E는 특정 CPU에 할당한 후 다른 job을 돌아다니게 함으로써 `affinity fairness`를 달성할 수도 있습니다.

요약하자면 구현은 간단하나 확장성과 `cache affinity`를 고려하기 어렵습니다.

## Multi-Queue Scheduling

다른 전략은 CPU마다 queue를 사용하는 Multi-Queue multiprocessor scheduling입니다. 줄여서 MQMS라고도 합니다. 작업이 할당되면 특정 휴리스틱(랜덤이나 작업이 작은 큐를 고름)에 따라 특정 큐에 배치하도록 합니다. 이를 통해 SQMS에서 발생했던 문제가 줄어들게 됩니다. 

<img width="338" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/c0ebb8ec-e04e-4e28-8598-d2a5fe39adaf">

두개의 큐에서 Round Robin 정책을 사용한다면 아래와 같이 스케줄링 될 것입니다.

<img width="530" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/5d629916-2e29-4101-bb21-8e72c7045962">

SQMS에서 문제가 되었던 확장성 문제는 CPU가 증가하면서 동시에 큐의 숫자도 증가하므로 문제가 되지 않을 뿐더러 자신의 큐에 있는 job만 처리하기 때문에 `cache affinity`도 고려하고 있습니다. 하지만 문제는 job C가 끝나면서 발생합니다.

<img width="630" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/a505641d-3b51-4f9e-8f48-e18d9a8ab70e">

이전 글에서 CPU시간을 공평하게 사용하자! 라고 이야기 했는데 위 상황에서는 A가 다른 job보다 CPU를 2배나 더 쓰고 있는 상황입니다. 심지어 여기서 A마저 끝난다면 CPU 0번에는 아무것도 스케줄링 되지 않는 최악의 상황까지 발생합니다. MQMS에서는 이러한 문제를 `load imbalance` , 즉 부하 불균형이라 부릅니다.

해결책은 무엇일까요? 간단합니다. job을 다른 CPU로 `migration`하면 됩니다. 하지만 위 사진처럼 CPU 1에 있는 job B를 CPU 0으로 한번 이동한다고 해서 불균형 문제가 해소되진 않습니다. 즉 주기적인 `migration`이 필요합니다.

<img width="768" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/9950aad3-9c6a-44c5-8da3-953beb688d93">

이제 각 job마다 CPU를 공평하게 나눠쓰고 있게 되었습니다. 이를 구현하는 방법 중 하나는 `work stealing`
입니다. 작업의 수가 적은 큐에서 작업이 많은 큐를 주기적으로 들여다보면서 하나 이상의 작업을 훔칩니다. 역시나 ***적절한*** 주기로 작업을 훔쳐야하는데 너무 빠른 주기로 훔치면 오버헤드가 커지고 너무 긴 주기로 훔치면 `load imbalance`가 발생하기 때문입니다. 글에서도 여러번 언급되지만 적절한 임계값을 찾는 건 매우 까다롭습니다.

## 참고

write policy cache
- https://electronic-hwan.tistory.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EA%B5%AC%EC%A1%B0-Write-ThroughWrite-Back-Allocate

cache coherence vs synchronization
-  https://stackoverflow.com/questions/68277817/why-do-we-need-the-volatile-keyword-when-the-core-cache-synchronization-is-done
