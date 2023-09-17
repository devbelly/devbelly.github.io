---
title: "Test"
date: 2023-09-17T12:13:59+09:00
draft: true
comments: true
toc: true
tags:
  - untagged
---
안녕하세요. 이번시간에는 Multiprocessor Scheduling에 대해 알아보겠습니다. 멀티프로세서는 두 개 이상의 CPU가 있는 컴퓨터 시스템을 가리키고 각각의 주변장치 뿐만 아니라 메인 메모리도 공유합니다. 일반적인 애플리케이션은 단일 CPU만 사용하므로 멀티프로세서 환경일지라도 처리속도가 빨라지진 않습니다. 이를 개선하기 위해 작성된 프로그램을 멀티 스레드를 사용하도록 재작성해야합니다. 멀티 스레드 애플리케이션은 여러 CPU에 작업을 분산할 수 있기 때문입니다. 

## Background: Multiprocessor Architecture

위처럼 애플리케이션 레벨의 문제말고도 OS수준에서도 문제가 발생합니다. 왜냐하면 지금까지 살펴본 스케줄링은 모두 단일 CPU를 가정했기 때문이죠. 

CPU는 1초에 여러 개의 instruction을 실행할 수 있습니다. 매 번 RAM에 접근해서 데이터를  읽어 온 후 CPU에서 instruction을 수행하는 것은 비효율적이므로 성능 향상을 위해 다양한 전략을 사용합니다. 그 중 하나가 캐싱입니다. 캐시는 계층 구조를 띠며(L1, L2, L3) 메인 메모리에서 ***인기 있는*** 데이터의 사본을 저장하는 작고 빠른 메모리입니다. 반면 메인 메모리는 모든 데이터를 보유하지만 해당 데이터에 대한 엑세스는 느립니다. 때문에 캐시를 활용하여 느린 메모리를 빠른 메모리 처럼 보이게 하는 효과를 얻게됩니다.

처음 프로그램이 실행되면 캐시에는 아무런 정보가 없고 원하는 데이터가 메인 메모리에 있으므로 데이터를 가져오는데 오래 걸립니다. 가져온 정보가 캐싱이 된 후 나중에 다시 동일한 데이터에 접근한다면 CPU는 먼저 캐시에서 해당 데이터를 찾게 되므로 훨씬 빠르게 접근할 수 있습니다. 

그렇다면 도대체 ***인기 있는*** 데이터가 무엇일까요? 이를 판단하기 위해 `locality`개념을 사용합니다. `locality`에는 `temporal locality`와 `spatial locality` 가 있습니다. `temporal locality`는 말그대로 시간과 관련된 `locality`로 어떤 데이터에 접근했다면 가까운 미래에 해당 데이터를 다시 접근한다는 의미입니다. `spatial locality`는 공간과 관련된 `locality`로 어떤 데이터에 접근했다면 해당 데이터와 메모리주소가 가까운 다른 데이터들도 접근한다는 의미입니다. 배열을 순회하는 프로그램이 이에 해당합니다.

<img width="486" alt="image" src="https://github.com/devbelly/image-issue/assets/67682840/0398e170-4fed-4310-bc95-0b28fbc9ea8e">

이제 CPU가 여럿인 상황에 대해 생각해볼까요? 각 CPU마다 고유의 캐시를 갖고 있습니다. 예를 들어 CPU 1에서 메모리 주소 A에 접근하고자 한다면 일단 캐시는 해당 데이터가 있는지 살펴봅니다. 해당 내용이 없으므로 캐시는 A에 접근하여 D라는 값을 읽고 프로그램을 수행하여 D’로 업데이트를 합니다. 이 때 CPU 2에서 메모리 주소 A에 접근한다면 D’이 아니라 D를 읽어오는 문제가 발생합니다. 

왜냐하면 `volatile`이 선언되지 않은 변수에 대해 즉시 메인 메모리에 변경사항을 반영하지 않기 때문입니다. CPU는 write request를 메모리에 즉시 반영하는 대신 write buffer에 request를 저장했다가 메인 메모리에 한번에 반영합니다. 

이러한 `cache coherence`문제를 해결하기 위한 하드웨어 수준에서의 해결책은 `snooping`이 있습니다. 

Don’t forget~

## 참고

write policy cache
- https://electronic-hwan.tistory.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EA%B5%AC%EC%A1%B0-Write-ThroughWrite-Back-Allocate
- 