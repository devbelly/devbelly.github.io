---
title: "Test"
date: 2023-09-21T09:07:24+09:00
draft: true
comments: true
toc: true
tags:
  - untagged
---
안녕하세요. 이번 시간에는 주소 변환에 대해 살펴보겠습니다. CPU virtualization를 위해서 OS는 효율성과 제어 두가지 측면을 고려해야 했습니다. 때문에 `Limited Direct Execution`을 개발했고 프로세스는 CPU 하드웨어에서 `Direct`하게 실행되지만 특정 시점에 OS가 개입하여 프로세스가 할 수 있는 동작에 `limited`를 두었습니다. 

마찬가지로 Memory Virtualization은 `hardware based address translation`, 줄여서 `address translation` 을 통해 구현가능합니다.  주소 변환을 사용하면 하드웨어가 각 메모리 엑세스를 변환하여 명령어가 제공하는 `Virtual Address`를 `Physical Address`로 변환해줍니다. 따라서 모든 메모리 참조에 대해 하드웨어에서 주소 변환을 수행하여 애플리케이션 메모리 참조를 메모리 내 실제 위치로 리디렉션합니다.

이번 글에서는 아래 과정을 어떻게 해결해나가는지 살펴보겠습니다.

- 효율성
- 제어
- 유연성

하드웨어를 거치면 효율적이긴 하다(사진)
Of course, the hardware alone cannot virtualize memory, as it just pro- vides the low-level mechanism for doing so efficiently. 

하드웨어만으론 VM을 제공할 수 없고 OS가 메모리들에 대해서 관리를 해주어야 최종적으로 VM을 제공할 수 있다.

Memory Virtualization을 통해서 프로세스가 메모리를 독점하고 있다는 환상을 제공하는 것이 최종 목표입니다. 소스코드를 작성하면서 이러한 환상에 대한 혜택을 누리고 있습니다. 내가 선언하는 배열이 다른 프로세스의 범위를 넘어설까 걱정하지 않기 때문입니다. 어쨋든 OS는 하드웨어의 도움을 받아 아름다운 가상의 메모리를 제공한다.

## Assumption

Memory Virtualization 어떻게 변화했는지 살펴보기 위해 아래와 같은 가정들을 하겠습니다.

- 사용자의 address spsace는 물리 메모리에 `contiguously` 하게 배치되어있다.
- address spacce의 크기가 물리 메모리 보다 작다
- 모든 address space의 크기는 일치한다.

스케줄링때와 마찬가지로 이 비현실적인 가정들을 없애나가며 최종적인 Memory virtualization에 대해 이해해보겠습니다.

## An Example

예제는 메모리에 하나으 ㅣ변수가 선언되어ㅣㅇㅆ고 증가하고 다시 저장하는 역할을 하는 코드
어셈블리로 살펴보면 다음과 같다.

~설명~

address space는 0에서 시작해 16KB까지 범위이다. 이를 가상화 하기 위해 OS는 PM에 옮겨놔야하고 

여기서 문제가 발생한다. 어떻게 프로세스가 모르는 채로 우리는 relocate를 할 수 있을까?
어떻게 VM은 0부터 시작하시만 PA는 다른 곳에서 시작할 수 있을까?



Limited Direct Execution
- https://www.chegg.com/flashcards/chapter-6-mechanism-limited-direct-execution-945601e8-979b-4afe-b4ee-6ea0b775e505/deck